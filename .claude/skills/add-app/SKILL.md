---
name: add-app
description: Use when adding a new application to the homelab Flux CD GitOps setup, scaffolding Helm-based deployments with HelmRepository, HelmRelease, ConfigMap values, SealedSecrets, and kustomization wiring.
---

# add-app

## Overview

Checklist-driven scaffold for a complete new app in the homelab Flux CD + Kustomize + Helm GitOps stack. Follow every step in order — skipping any step causes silent GitOps failures.

## Checklist

### Step 1 — HelmRepository (skip if chart source already exists)

File: `infrastructure/controllers/<name>-helmrepo.yaml`

**HTTP chart repo** (no `type` field):
```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: <name>
  namespace: flux-system
spec:
  interval: 1h
  url: https://<chart-repo-url>
```

**OCI registry** (add `type: "oci"`):
```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: <name>
  namespace: flux-system
spec:
  type: "oci"
  interval: 1h
  url: oci://<registry-url>
```

Then add the file to `infrastructure/controllers/kustomization.yaml` resources list.

---

### Step 2 — App directory + namespace

Create `apps/<name>/namespace.yaml`:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <name>
```

---

### Step 3 — HelmRelease

File: `apps/<name>/<name>-helmrelease.yaml`:
```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: <name>
  namespace: <name>
spec:
  interval: 1h
  chart:
    spec:
      chart: <chart-name>
      version: "<semver-range>"
      sourceRef:
        kind: HelmRepository
        name: <helmrepo-name>   # must match HelmRepository metadata.name
        namespace: flux-system
      interval: 1h
  install:
    remediation:
      retries: 3
  upgrade:
    remediation:
      retries: 3
  valuesFrom:
    - kind: ConfigMap
      name: <name>-values
    # add below only if using sealed secrets:
    # - kind: Secret
    #   name: <name>-secret-values
```

Semver range conventions used in this project: `"16.x"` (major pinned), `"2.x"` (major pinned), `"1.8.x"` (major+minor pinned).

**Finding the current major version:** Check the chart's GitHub releases page (e.g. `https://github.com/grafana/helm-charts/releases?q=grafana-`) or ArtifactHub. The chart version and the app version are often different — use the chart version for the semver range. Known versions as of 2026-03: `prometheus-community/prometheus` → `28.x`, `grafana/grafana` → `10.x`.

---

### Step 4 — Values ConfigMap

File: `apps/<name>/<name>-values.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: <name>-values
  namespace: <name>
data:
  values.yaml: |
    # non-sensitive Helm values here
    # e.g. ingress hosts, persistence, resources
```

The data key **must** be exactly `values.yaml:`. Sensitive values (passwords, API keys) go in a SealedSecret instead.

---

### Step 5 — Sealed Secret (only if sensitive values needed)

**Invoke the `seal-secret` skill** — do not run kubeseal locally. The cluster cert is only accessible server-side and the `johan` user requires `KUBECONFIG=/home/johan/.kube/config` for kubectl. The seal-secret skill handles all of this correctly.

The secret name must be `<name>-secret-values` and live in the `<name>` namespace. Its `values.yaml` key contains the sensitive Helm values that would otherwise go in the ConfigMap.

Then add to HelmRelease `valuesFrom`:
```yaml
- kind: Secret
  name: <name>-secret-values
```

---

### Step 6 — App kustomization.yaml

File: `apps/<name>/kustomization.yaml`:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - <name>-helmrelease.yaml
  - <name>-values.yaml
  # - <name>-sealed-secret.yaml   # add if created in step 5
```

---

### Step 7 — Register app in apps/kustomization.yaml

Add `- <name>` to the resources list in `apps/kustomization.yaml`.

---

### Step 8 — DNS entry (optional)

Add to `customDnsEntries` in `apps/pihole/pihole-values.yaml`:
```yaml
- address=/<name>.lan/192.168.1.205
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting to update `infrastructure/controllers/kustomization.yaml` after adding a HelmRepository | Always update the kustomization resources list in the same commit |
| Wrong namespace in HelmRelease `metadata.namespace` | Must match the namespace created in `namespace.yaml` |
| Sensitive values (passwords, API keys) placed in ConfigMap | Move to SealedSecret; ConfigMaps are stored plaintext in git |
| Wrong `sourceRef.name` in HelmRelease | Must exactly match `metadata.name` of the HelmRepository |
| Forgetting to add app dir to `apps/kustomization.yaml` | Flux will never reconcile the app without this entry |
| Using `type: "oci"` for HTTP repos or omitting it for OCI | HTTP repos: no type field; OCI registries: `type: "oci"` required |
