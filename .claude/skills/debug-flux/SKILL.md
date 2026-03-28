---
name: debug-flux
description: Use when a HelmRelease is failing, stuck reconciling, or an app is not deploying. Always use this skill when any deployed app stops working, shows errors in Flux, is stuck in Reconciling, throws install retries exhausted, or isn't showing up after a git push — Claude SSHes in, diagnoses the error, and applies fixes directly.
---

# debug-flux

## Overview

Claude SSHes to `johan@192.168.1.205`, runs triage commands, interprets the output, and applies fixes — all via Bash tool calls. The user describes the symptom; Claude handles the rest.

**KUBECONFIG:** The `johan` user cannot read `/etc/rancher/k3s/k3s.yaml`. Prefix every `kubectl` and `flux` command with `KUBECONFIG=/home/johan/.kube/config`. Without this, commands fail silently with a permission error.

---

## 1 — Triage loop

Claude runs these commands via SSH to identify the failing resource:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config flux get all -A"
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl get helmreleases -A"
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config flux logs --all-namespaces --level=error --tail=100"
```

Claude reads the output and identifies which HelmRelease (name + namespace) is failing and what the error message says.

---

## 2 — Diagnosis

Claude runs a detailed describe on the failing resource:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl describe helmrelease <name> -n <ns>"
```

Claude matches the error text against the error reference below to determine the fix.

---

## 3 — Error reference and fixes

| Error text | Cause | Fix Claude applies |
|------------|-------|-------------------|
| `chart fetch failed` | Bad HelmRepository URL or missing OCI type | Read the HelmRepository YAML in the repo, identify the error, fix and commit |
| `install retries exhausted` | Helm install failing at pod level | SSH in, run `kubectl logs -n <ns>` on the failing pod, surface findings to user |
| `values schema validation failed` | ConfigMap values don't match chart schema | Read `apps/<name>/<name>-values.yaml`, identify bad keys, fix the ConfigMap |
| `dependency not ready` | Infrastructure HelmReleases not healthy | Run `flux get helmreleases -n flux-system` and check longhorn/sealed-secrets |
| `SealedSecret decryption error` | Secret not decrypting | Invoke `seal-secret` skill to re-seal |
| `stuck in Reconciling` | HelmRelease needs a nudge | Run `flux reconcile helmrelease <name> -n <ns>`, or delete the HelmRelease and let Flux recreate it |
| `ImagePullBackOff` or `ErrImagePull` | Cluster can't pull container image | Check `imagePullSecrets` on the pod, verify the registry SealedSecret exists in the correct namespace (`kubectl get sealedsecret -n <ns>`), re-seal if missing |

---

## 4 — Fix execution

After identifying the cause, Claude applies the appropriate fix:

**Force reconcile:**
```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config flux reconcile helmrelease <name> -n <ns> --with-source"
```

**Delete and recreate (for stuck resources):**
```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl delete helmrelease <name> -n <ns>"
# Flux will recreate it on next reconcile — trigger immediately:
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config flux reconcile kustomization apps --with-source"
```

**Check pod logs for install failures:**
```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl get pods -n <ns>"
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl logs -n <ns> <pod-name> --previous"
```

---

## 5 — Post-fix verification

After applying any fix, verify the HelmRelease is healthy:
```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl get helmrelease <name> -n <ns>"
```

Expected: `READY: True`, `STATUS: Release reconciliation succeeded`.

If still failing after 2 minutes, re-run the triage loop (Section 1). For pod-level crashes not caused by Helm config, invoke the `ssh-debug` skill.

---

## Common Mistakes

| Mistake | What Claude should watch out for |
|---------|----------------------------------|
| Fixing the wrong namespace | HelmReleases live in the app namespace (e.g. `forgejo`), not `flux-system` |
| Reconciling kustomization instead of helmrelease | For a single failing app, reconcile the specific HelmRelease, not all apps |
| Editing `clusters/main/flux-system/` files | Those are Flux bootstrap files — never edit them manually |
| Assuming the issue is in the chart when it's in the values | Check the ConfigMap values against the chart's `values.yaml` before blaming the chart |
| Deleting a pod to fix a HelmRelease error | Pod restarts don't fix Helm state — reconcile the HelmRelease instead |
| Not checking infrastructure HelmReleases when apps fail | Apps depend on `infrastructure-controllers` (longhorn, sealed-secrets) being healthy |
| Running `kubectl` or `flux` without KUBECONFIG | `/etc/rancher/k3s/k3s.yaml` is not readable by `johan`; always prefix with `KUBECONFIG=/home/johan/.kube/config` |
