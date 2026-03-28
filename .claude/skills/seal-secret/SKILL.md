---
name: seal-secret
description: Use when creating, updating, or verifying a SealedSecret for a homelab app. Always use this skill whenever the task involves secrets, passwords, API keys, database credentials, tokens, or any sensitive config that needs to be encrypted before committing to git — Claude SSHes to the server and runs kubeseal directly.
---

# seal-secret

## Overview

Claude SSHes to `johan@192.168.1.205`, runs the `kubectl | kubeseal` pipeline there, reads back the sealed YAML, and writes it to the repo. The user never runs kubeseal manually.

**Conventions:**
- Controller: `sealed-secrets-controller` in `kube-system`
- Secret name: `<name>-secret-values`
- Secret namespace: `<name>` (same as the app namespace)
- Output file: `apps/<name>/<name>-sealed-secret.yaml`
- Referenced from HelmRelease as `kind: Secret, name: <name>-secret-values`

**KUBECONFIG:** The `johan` user cannot read `/etc/rancher/k3s/k3s.yaml` (permission denied). Always use `KUBECONFIG=/home/johan/.kube/config` for `kubectl` and `--kubeconfig /home/johan/.kube/config` for `kubeseal`. Without this, both commands fail with a permission error and produce no output.

---

## 1 — Create a new SealedSecret

Claude runs via Bash tool (SSH):

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl create secret generic <name>-secret-values \
  -n <name> \
  --dry-run=client \
  --from-literal=<KEY>=<VALUE> \
  -o yaml \
  | kubeseal --kubeconfig /home/johan/.kube/config -o yaml"
```

**Multiple keys:** Chain `--from-literal` flags: `--from-literal=KEY1=VAL1 --from-literal=KEY2=VAL2`.

**Helm `values.yaml` key** (entire Helm values block as one secret key): Write the values to a temp file on the server, then seal:
```bash
ssh johan@192.168.1.205 "cat > /tmp/secret-vals.yaml << 'EOF'
adminPassword: "mypassword"
apiKey: "mykey"
EOF
KUBECONFIG=/home/johan/.kube/config kubectl create secret generic <name>-secret-values \
  -n <name> --dry-run=client \
  --from-file=values.yaml=/tmp/secret-vals.yaml \
  -o yaml | kubeseal --kubeconfig /home/johan/.kube/config -o yaml"
```

Claude then:
1. Reads the sealed YAML output from the command
2. Writes it to `apps/<name>/<name>-sealed-secret.yaml` in the repo
3. Adds the file to `apps/<name>/kustomization.yaml` resources list
4. Adds the `valuesFrom` entry to the HelmRelease:

```yaml
valuesFrom:
  - kind: ConfigMap
    name: <name>-values
  - kind: Secret
    name: <name>-secret-values
```

---

## 2 — Update an existing SealedSecret

**Warning:** Sealed secrets cannot be partially updated. Claude must include ALL keys when re-sealing — omitting a key removes it.

Claude re-runs the full `kubectl | kubeseal` pipeline with all key-value pairs, then overwrites `apps/<name>/<name>-sealed-secret.yaml`.

---

## 3 — Verify decryption succeeded

After committing and reconciling, Claude SSHes in and checks:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl get sealedsecret <name>-secret-values -n <name>"
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl get secret <name>-secret-values -n <name>"
```

If the `Secret` exists, decryption succeeded. Claude also checks the pod is consuming it:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl describe pod -n <name> | grep -A5 envFrom"
```

---

## 4 — Registry pull secret (docker-registry type)

Used to allow the cluster to pull images from a private registry. The secret type is `docker-registry`, not `generic`.

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl create secret docker-registry <name>-registry-secret \
    --docker-server=forgejo.lan \
    --docker-username=kvarnberg \
    --docker-password=<token> \
    --namespace=<name> \
    --dry-run=client -o yaml \
  | kubeseal --kubeconfig /home/johan/.kube/config -o yaml"
```

Reference from Deployments as `imagePullSecrets`, not `envFrom` or `valuesFrom`:
```yaml
spec:
  imagePullSecrets:
    - name: <name>-registry-secret
```

**The namespace must exist before kubeseal can run** — apply it manually if needed.

---

## Common Mistakes

| Mistake | What Claude should watch out for |
|---------|----------------------------------|
| Re-sealing with only changed keys | Always include ALL keys — omitting any key deletes it from the decrypted secret |
| Wrong namespace in `kubectl create secret` | The `-n` flag must match the app namespace, not `kube-system` or `flux-system` |
| Forgetting to add sealed secret to `kustomization.yaml` | Flux won't apply the file unless it's listed in the app's kustomization resources |
| Forgetting to add `valuesFrom` entry in HelmRelease | The secret won't be passed to Helm even if it exists in Kubernetes |
| Plaintext values left in ConfigMap after moving to SealedSecret | Remove them from the ConfigMap — they must not exist in both places |
| Encrypting with wrong cluster cert | `kubeseal` on the server uses the local cluster's cert — always run it on the node, not locally |
| Running `kubectl` or `kubeseal` without KUBECONFIG | `/etc/rancher/k3s/k3s.yaml` is not readable by `johan`; omitting `KUBECONFIG=/home/johan/.kube/config` produces a silent permission error with no output |
