# Eval 1 — act-runner stuck Init:0/1

## Prompt
The act-runner pod in the act-runner namespace is stuck in Init:0/1 — it's been like this for several minutes. Figure out what's wrong and fix it.

## Root cause
`.lan` hostname not resolvable inside the k3s cluster. The init container was configured with `--instance http://forgejo.lan` which fails from within any pod. Fix: use `http://forgejo-http.forgejo.svc.cluster.local:3000`.

## Skill guidance followed
- Step 1: located pod via `kubectl get pods -n act-runner`
- Step 2: `kubectl describe pod` — Events showed `BackOff restarting failed container register`
- Step 3 — Init:X/N branch: fetched init container logs, identified DNS failure
- Step 4: updated deployment.yaml with in-cluster URL + `generate-config` step + `fsGroup: 1000`
- Step 5: committed, pushed, `flux reconcile kustomization apps --with-source` → pod reached 2/2 Running

## Skill improvement identified
Init container table should include an explicit row for `could not resolve host` / `Name or service not known`.
