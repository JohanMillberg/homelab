# Homelab GitOps

Flux CD + Kustomize + Helm GitOps repository for a single Kubernetes cluster (`clusters/main`).

## Repository Structure

```
clusters/main/          # Flux entrypoint — defines what gets reconciled
  infrastructure.yaml   # Deploys infrastructure/controllers then infrastructure/configs
  apps.yaml             # Deploys apps/ (depends on infrastructure-configs)
  flux-system/          # Flux bootstrap manifests (don't edit manually)

infrastructure/
  controllers/          # HelmRepositories + core HelmReleases (longhorn, sealed-secrets)
  configs/              # Cluster-wide config (currently empty)

apps/                   # One subdirectory per app
  kustomization.yaml    # Register new apps here
  <name>/
    namespace.yaml
    <name>-helmrelease.yaml
    <name>-values.yaml          # ConfigMap with non-sensitive Helm values
    <name>-sealed-secret.yaml   # SealedSecret for sensitive values (if needed)
    kustomization.yaml
```

## Reconciliation Order

`infrastructure-controllers` → `infrastructure-configs` → `apps`

Flux polls git every 1h. To apply changes immediately:
```bash
flux reconcile kustomization apps --with-source
flux reconcile kustomization infrastructure-controllers --with-source
```

## Current Apps

| App | Namespace | Chart source |
|-----|-----------|--------------|
| pihole | pihole | mojo2600 (HTTP) |
| forgejo | forgejo | OCI `code.forgejo.org/forgejo-helm/` |
| n8n | n8n | OCI `8gears.container-registry.com/library` |
| grafana | grafana | grafana (HTTP, grafana.github.io/helm-charts) |
| prometheus | prometheus | prometheus-community (HTTP) |
| plan-assistant | plan-assistant | Raw manifests (FastAPI + Vue.js + PostgreSQL) — image: `forgejo.lan/kvarnberg/plan-assistant-{backend,frontend}` |

Infrastructure: longhorn (storage), sealed-secrets (secret encryption).

**Chart version → App version notes (as of 2026-03):**
- `grafana/grafana` chart `10.x` → Grafana app **12.x** (supports `grafanactl`)
- `prometheus-community/prometheus` chart `28.x`

## Key Conventions

- **HelmRelease `apiVersion`**: always `helm.toolkit.fluxcd.io/v2`
- **Semver ranges**: `"16.x"` or `"2.x"` (major pinned) preferred over exact pins
- **Values ConfigMap**: data key must be exactly `values.yaml:`
- **Sensitive values**: always in SealedSecret, never in ConfigMap
- **HelmRepository namespace**: always `flux-system`
- **OCI repos**: require `type: "oci"` and `oci://` URL; HTTP repos omit `type`
- **DNS**: internal hostnames follow `<name>.lan`, resolved via pihole at `192.168.1.205` — **note:** the node itself uses 1.1.1.1 for DNS, so `.lan` hostnames do not resolve in SSH commands; use cluster IPs or `kubectl port-forward` instead

## Skills

Use the skills in `.claude/skills/` for common tasks:

- **`add-app`** — scaffold a complete new app (HelmRepo → HelmRelease → values → kustomization wiring) — Helm-based only
- **`deploy-raw-app`** — deploy a non-Helm app with raw Kubernetes manifests and self-built Docker images pushed to Forgejo
- **`upgrade-app`** — upgrade a chart version or pinned image tag
- **`seal-secret`** — create, update, or verify a SealedSecret (Claude runs kubeseal on the server)
- **`debug-flux`** — diagnose and fix stuck HelmReleases (Claude SSHes in and applies fixes)
- **`ssh-debug`** — full node-level triage via SSH (Claude runs diagnostics and fixes on the server)
- **`manage-grafana-dashboards`** — add, update, or verify a Grafana dashboard (JSON → kustomization wiring → grafanactl verification)

## Sealed Secrets Workflow

Secrets are encrypted with `kubeseal` before committing. The controller runs as `sealed-secrets-controller` in `kube-system`.

```bash
# Create a sealed secret for app values
kubectl create secret generic <name>-secret-values \
  -n <name> \
  --dry-run=client \
  -o yaml \
  | kubeseal -o yaml > apps/<name>/<name>-sealed-secret.yaml
```

Reference from HelmRelease `valuesFrom` as `kind: Secret, name: <name>-secret-values`.

## Do Not

- Edit files under `clusters/main/flux-system/` — managed by Flux bootstrap
- Put plaintext secrets or passwords in ConfigMaps or any unencrypted file
- Add a new app directory without also registering it in `apps/kustomization.yaml`
- Add a new HelmRepository without also updating `infrastructure/controllers/kustomization.yaml`
