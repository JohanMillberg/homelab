---
name: upgrade-app
description: Use when upgrading a homelab app's Helm chart version, pinned image tag, or both. Always use this skill when someone says "upgrade", "bump version", "update chart", "newer release", or mentions wanting a newer version of any app (forgejo, n8n, grafana, prometheus, pihole, etc.) — guides which file to edit, semver range conventions, and schema-change checks.
---

# upgrade-app

## Overview

Three upgrade cases for apps in this Flux CD + Kustomize + Helm homelab. Identify the case, edit the right file, and follow the checks.

## Case A — Upgrade Helm chart version

File: `apps/<name>/<name>-helmrelease.yaml`

Update `spec.chart.spec.version`. Use the semver range conventions this project follows:

| Range | Example | When to use |
|-------|---------|-------------|
| `"X.x"` | `"16.x"`, `"2.x"` | Major pinned, allow minor+patch updates |
| `"X.Y.x"` | `"1.8.x"` | Major+minor pinned, allow patch updates only |
| `"X.Y.Z"` | `"2.11.2"` | Exact pin (use sparingly, only when necessary) |

**Before bumping a major version:** Check the chart's changelog — values schema often changes between major versions (renamed keys, removed fields, new required values).

For OCI charts: same range syntax applies.

---

## Case B — Upgrade pinned image tag

File: `apps/<name>/<name>-values.yaml`

Update the `image.tag` (or equivalent) in the `data.values.yaml` block.

Example (n8n):
```yaml
data:
  values.yaml: |
    image:
      repository: n8nio/n8n
      tag: "2.11.3"   # ← update this
```

**Note:** Only n8n currently pins an image tag. Most apps rely on chart-bundled images and don't need this step.

---

## Case C — Both chart version and image tag

1. Update `<name>-helmrelease.yaml` version first (Case A)
2. Check if the new chart version changes the values schema (e.g., `image.tag` path renamed)
3. Update `<name>-values.yaml` accordingly (Case B)

---

## Reconcile

After committing and pushing:
```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config flux reconcile kustomization apps --with-source"
```

Verify upgrade succeeded:
```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl get helmrelease <name> -n <name>"
```

Expected: `READY: True`, `STATUS: Release reconciliation succeeded`. If it fails, invoke the `debug-flux` skill.

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Editing the values ConfigMap instead of the HelmRelease for a chart upgrade | Chart version lives in `<name>-helmrelease.yaml` under `spec.chart.spec.version` |
| Editing the HelmRelease for an image tag bump | Image tag lives in `<name>-values.yaml` in the ConfigMap data block |
| Pinning exact version when a semver range is appropriate | Use `"X.x"` ranges unless you have a specific reason to pin exactly |
| Not checking values schema after a major chart bump | Values keys can change between major versions — reconciliation will fail silently |
| Expecting immediate rollout after a commit | Flux polls every 1h; always force reconcile after pushing: `ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config flux reconcile kustomization apps --with-source"` |
