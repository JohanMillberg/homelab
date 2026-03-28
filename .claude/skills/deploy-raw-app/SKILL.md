---
name: deploy-raw-app
description: Use when deploying an application to the homelab cluster using raw Kubernetes manifests instead of Helm. Always use this skill for apps with a custom Dockerfile, self-built images, or anything not available as a Helm chart — FastAPI backends, Vue/React frontends, custom Python/Node services, or anything pushed to the Forgejo registry at forgejo.lan. Covers StatefulSets, Deployments, Ingress, SealedSecrets, and image push workflow.
---

# deploy-raw-app

## Overview

Checklist for deploying a non-Helm app with raw Kubernetes manifests managed by Flux kustomization. Unlike `add-app`, there is no HelmRelease — you write Deployments/Services/Ingress directly and push Docker images to the Forgejo registry at `forgejo.lan`.

---

## Checklist

### Step 1 — Build and push Docker images

**Registry facts:**
- URL: `forgejo.lan` (HTTP only — Traefik serves HTTPS with a self-signed cert)
- Username: `kvarnberg` (lowercase — Docker registry requires lowercase)
- Login: `docker login forgejo.lan --username kvarnberg --password <token>`

**Critical build flag — Forgejo rejects OCI manifest lists (405 Not Allowed):**
```bash
docker buildx build \
  --output type=docker \
  --provenance=false \
  --sbom=false \
  -t forgejo.lan/kvarnberg/<app>:<tag> \
  -f Dockerfile .

docker push forgejo.lan/kvarnberg/<app>:<tag>
```

**Image tag convention** (Flux image automation compatible): `main-<git-sha>`

```bash
GIT_SHA=$(git -C <source-dir> rev-parse --short HEAD)
TAG="main-${GIT_SHA}"
```

If Docker Desktop is used on Windows: add `forgejo.lan` to insecure-registries via Settings → Docker Engine (UI-based; editing `~/.docker/daemon.json` alone has no effect).

---

### Step 2 — Create app manifests

Minimum files in `apps/<name>/`:

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace |
| `configmap.yaml` | Non-sensitive env vars |
| `<resource>-deployment.yaml` | Deployment(s) |
| `services.yaml` | Services |
| `ingress.yaml` | Traefik ingress (host: `<name>.lan`) |
| `kustomization.yaml` | Lists all resources |

**Reference implementation:** See `apps/plan-assistant/` for a complete working example (FastAPI backend + Vue.js frontend + PostgreSQL StatefulSet).

**imagePullSecrets** must reference the registry SealedSecret in every Deployment:
```yaml
spec:
  imagePullSecrets:
    - name: forgejo-registry-secret
```

**DATABASE_URL pattern:** Put the URL without password in the ConfigMap. Put the full URL (with password) in the SealedSecret. When both ConfigMap and Secret are in `envFrom`, the Secret's value wins (later source overrides earlier).

---

### Step 3 — Create registry pull SealedSecret

The cluster must be able to pull from `forgejo.lan`. Run on the server:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl create secret docker-registry forgejo-registry-secret \
    --docker-server=forgejo.lan \
    --docker-username=kvarnberg \
    --docker-password=<token> \
    --namespace=<name> \
    --dry-run=client -o yaml \
  | kubeseal --kubeconfig /home/johan/.kube/config -o yaml"
```

Write output to `apps/<name>/registry-sealed-secret.yaml`.

**The namespace must exist before kubeseal runs.** Apply it manually first if Flux hasn't reconciled yet:
```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl apply -f -" << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: <name>
EOF
```

---

### Step 4 — Create app SealedSecret

Use the `seal-secret` skill. For raw manifest apps, the secret uses `envFrom` (not `valuesFrom`) — name it `<name>-secret` (not `<name>-secret-values`).

---

### Step 5 — Register in apps/kustomization.yaml

Add `- <name>` to `apps/kustomization.yaml` resources list.

---

### Step 6 — DNS entry

Add to `customDnsEntries` in `apps/pihole/pihole-values.yaml`:
```yaml
- address=/<name>.lan/192.168.1.205
```

---

### Step 7 — Commit, push, reconcile

```bash
git add apps/<name>/ apps/kustomization.yaml apps/pihole/pihole-values.yaml
git commit -m "feat: deploy <name> app"
git push
```

Force immediate reconciliation:
```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config flux reconcile kustomization apps --with-source"
```

---

## Database migrations (Alembic / Python apps)

If the app uses Alembic with a `migrate` init container:

**Critical:** Alembic migration history must be complete from scratch. If the app was developed with SQLite + `create_all()`, there is no initial migration for the base tables. Fresh PostgreSQL has no tables, so any migration that `ALTER TABLE`s a pre-existing table will fail with `relation "<table>" does not exist`.

**Fix:** Create a new migration with `down_revision = None` that creates all base tables. Update the old base migration's `down_revision` to point to it.

**Also watch for:** `sa.DATETIME()` is SQLite-specific. Use `sa.DateTime()` for PostgreSQL compatibility.

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Docker push returns `405 Not Allowed` on manifest PUT | Use `--output type=docker --provenance=false --sbom=false` — Forgejo rejects OCI manifest lists |
| `ImagePullBackOff` on pods | Registry pull SealedSecret missing, wrong namespace, or wrong credentials |
| `relation "<table>" does not exist` in Alembic | Missing initial migration — base tables were created via `create_all()` in dev, never via Alembic |
| `type "datetime" does not exist` | `sa.DATETIME()` is SQLite-only; use `sa.DateTime()` |
| Namespace missing when running kubeseal | Apply namespace manually before sealing — kubeseal needs the namespace to exist |
| Docker login fails with TLS error | Add `forgejo.lan` to insecure-registries in Docker Engine settings (Docker Desktop UI, not `daemon.json`) |
| Wrong Forgejo username in image tags | Use `kvarnberg` (lowercase) — case-sensitive in registry paths |
| Missing `imagePullSecrets` on a Deployment | Every Deployment pulling from `forgejo.lan` needs `spec.imagePullSecrets: [{name: forgejo-registry-secret}]` |
| ConfigMap key duplicated in Secret `envFrom` | The Secret value wins (envFrom: ConfigMap first, Secret overrides) — remove the key from the ConfigMap to avoid confusion |
