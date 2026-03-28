---
name: debug-live-image
description: Use immediately when a pod won't start after a new Docker image is pushed or a deployment is updated on the homelab k3s cluster. Always trigger this skill for ImagePullBackOff, CrashLoopBackOff, Init:X/N stuck, OOMKilled, Pending, or 0/1 Ready — especially during the build-push-deploy loop of AI-driven development. Also use this when a rollout stalls, pods hang in Terminating, or a deployment never reaches the desired replica count after a code change.
---

# debug-live-image

## Overview

Claude SSHes to `johan@192.168.1.205`, finds the failing pod, reads the right logs, classifies the failure, and applies fixes — optimized for the fast "write code → push image → something broke" cycle of AI-driven development.

**KUBECONFIG:** Every `kubectl`/`flux` command must be prefixed with `KUBECONFIG=/home/johan/.kube/config` — the `johan` user cannot read `/etc/rancher/k3s/k3s.yaml`.

**DNS note:** `.lan` hostnames resolve from the dev machine but NOT from inside the k3s cluster (node or pods). Init containers and app containers must use in-cluster service URLs (`<service>.<namespace>.svc.cluster.local`) to reach other cluster services. Never use `forgejo.lan` in Deployment specs — use `http://forgejo-http.forgejo.svc.cluster.local:3000` instead.

---

## Step 1 — Locate the failing pod(s)

If the user named a specific namespace or deployment, start there. Otherwise scan everything:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl get pods -A | grep -vE '\bRunning\b|\bCompleted\b'"
```

Pick any pod(s) with a non-healthy status and carry them forward.

---

## Step 2 — Describe the pod

This is the fastest path to the root cause — Events at the bottom tell you *why* before you dig into logs:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl describe pod <pod> -n <ns> 2>&1 | tail -50"
```

Match the status and events against Step 3 to pick the right diagnosis branch.

---

## Step 3 — Classify → diagnose → fix

### ImagePullBackOff / ErrImagePull

The cluster can't fetch the image. Common in AI-driven dev when the tag was just pushed but the pull secret is wrong, missing, or the tag doesn't exist yet.

```bash
# Does the pull secret exist in this namespace?
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl get sealedsecret -n <ns>"

# Does the image tag actually exist in Forgejo? (run from dev machine, not over SSH)
curl -s "http://forgejo.lan/api/v1/repos/Kvarnberg/<app>/tags" \
  -H "Authorization: token <forgejo-token>" | python3 -m json.tool | grep name | head -5
```

**Fixes:**
- Missing pull secret → run `deploy-raw-app` skill Step 3 to create the SealedSecret
- Wrong tag in Deployment → update the image tag in the manifest, commit, push
- Registry unreachable from cluster → test from inside a running pod:
  `kubectl exec -n default <any-running-pod> -- wget -q -O- http://forgejo.lan`

---

### Init:X/N — init container stuck or crashing

The main container never starts until all init containers succeed. Identify which one is failing first:

```bash
# See which init containers exist and their states
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl get pod <pod> -n <ns> -o jsonpath='{range .status.initContainerStatuses[*]}{.name}: {.state}{\"\\n\"}{end}'"

# Fetch logs from the stuck init container
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl logs <pod> -n <ns> -c <init-container-name> 2>&1 | tail -60"
```

| Pattern in logs | Cause | Fix |
|-----------------|-------|-----|
| `Waiting for PostgreSQL...` loops | Postgres not running or wrong hostname | `kubectl get pods -n <ns>` — check postgres pod; verify service name in ConfigMap |
| `alembic` / `relation does not exist` | Missing initial migration (app used `create_all()` in dev) | Add a `down_revision=None` migration that creates base tables |
| `connection refused` on startup | Dependency service not running or wrong URL | Check env vars in ConfigMap match actual service names |
| `no such host: forgejo.lan` or DNS lookup failure | `.lan` hostname used inside a pod or init container | Use in-cluster URL: `http://forgejo-http.forgejo.svc.cluster.local:3000` |
| Pod pulling image — no logs yet | Still downloading; can be slow for large images | Wait and watch `kubectl describe pod` Events for pull progress |
| `exec: <command> not found` | Wrong command in init container spec | Fix the `command:` in the Deployment manifest |

---

### CrashLoopBackOff — main container crashes on startup

The previous run's logs (captured before the restart) usually contain the real error:

```bash
# The crash that just happened
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl logs <pod> -n <ns> --previous 2>&1 | tail -80"

# Current attempt (may not have much yet)
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl logs <pod> -n <ns> 2>&1 | tail -40"
```

Look for: missing environment variables, import errors from a bad dependency, failed DB connection string, or an exception during startup. Fix in code or env/config, build a new image, and push.

---

### 0/1 Ready — pod Running but readiness probe keeps failing

The container started but the app isn't responding to the probe. Check what the probe expects vs. what the app serves:

```bash
# See probe config
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl describe pod <pod> -n <ns> | grep -A8 'Readiness'"

# Watch app logs during startup
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl logs <pod> -n <ns> --follow --since=60s 2>&1 | head -60"
```

If the path/port are correct but the app is slow to start, increase `initialDelaySeconds` in the Deployment. If the app is throwing errors, those logs will show the runtime issue.

---

### OOMKilled — container exceeds memory limit

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl top pod <pod> -n <ns>"
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl describe pod <pod> -n <ns> | grep -A4 'Limits\|OOMKilled'"
```

Raise the `resources.limits.memory` in the Deployment manifest and redeploy.

---

### Pending — pod not scheduled

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl describe pod <pod> -n <ns> | grep -A10 'Events'"
```

| Event message | Cause | Fix |
|---------------|-------|-----|
| `unbound PersistentVolumeClaim` | PVC not yet provisioned | `kubectl get pvc -n <ns>` — if Pending, Longhorn may need a moment; check Longhorn is healthy |
| `Insufficient memory/cpu` | Node over-committed | `kubectl top nodes` — remove unused pods or increase requests |
| `node(s) had untolerated taint` | Node selector/toleration mismatch | Compare pod spec tolerations vs. `kubectl describe node` taints |

---

## Step 4 — Apply fix and verify

After committing a manifest change to the homelab repo and pushing, force immediate reconciliation:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config flux reconcile kustomization apps --with-source"
```

For a code/image fix (new image tag): update the image tag in the manifest, commit, push, reconcile.

For a quick restart without a manifest change (e.g., transient failure):

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl rollout restart deployment/<name> -n <ns>"
```

Wait for the pod to reach Running/Ready:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl get pods -n <ns> -w --request-timeout=90s"
```

---

## Common Mistakes

| Mistake | What to watch for |
|---------|-------------------|
| Reading current logs for CrashLoopBackOff | Current logs only show the start of the new attempt; `--previous` has the actual crash |
| Skipping `describe` and going straight to logs | Events reveal pull failures and scheduling issues before the container even starts |
| Confusing init container failure with main container failure | An init failure blocks the main container; always check `initContainerStatuses` first |
| Using `.lan` hostnames in Deployment/init container specs | `.lan` DNS doesn't resolve inside pods; use `<svc>.<ns>.svc.cluster.local` — e.g., `http://forgejo-http.forgejo.svc.cluster.local:3000` |
| Running curl over SSH to test `.lan` connectivity | `.lan` DNS doesn't resolve on the k3s node itself; use `kubectl exec` into a running pod |
| Reconciling all `apps` when only one pod is broken | For faster iteration, `kubectl rollout restart` skips Flux and is immediate |
| Fetching logs from the wrong container | Multi-container pods (e.g., DinD sidecar) need `-c <container-name>`; omitting it gets only the first container |
