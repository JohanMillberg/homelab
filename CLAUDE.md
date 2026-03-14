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

Infrastructure: longhorn (storage), sealed-secrets (secret encryption).

## Key Conventions

- **HelmRelease `apiVersion`**: always `helm.toolkit.fluxcd.io/v2`
- **Semver ranges**: `"16.x"` or `"2.x"` (major pinned) preferred over exact pins
- **Values ConfigMap**: data key must be exactly `values.yaml:`
- **Sensitive values**: always in SealedSecret, never in ConfigMap
- **HelmRepository namespace**: always `flux-system`
- **OCI repos**: require `type: "oci"` and `oci://` URL; HTTP repos omit `type`
- **DNS**: internal hostnames follow `<name>.lan`, resolved via pihole at `192.168.1.205`

## Skills

Use the skills in `.claude/skills/` for common tasks:

- **`add-app`** — scaffold a complete new app (HelmRepo → HelmRelease → values → kustomization wiring)
- **`upgrade-app`** — upgrade a chart version or pinned image tag

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
