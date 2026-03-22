# Forgejo Registry Pull Secret

Run the following command to create and seal the imagePullSecret for pulling
images from the Forgejo container registry:

```bash
kubectl create secret docker-registry forgejo-registry-secret \
  --docker-server=forgejo.lan \
  --docker-username=YOUR_FORGEJO_USERNAME \
  --docker-password=YOUR_FORGEJO_TOKEN_OR_PASSWORD \
  --namespace=plan-assistant \
  --dry-run=client -o yaml \
  | kubeseal --format yaml > registry-sealed-secret.yaml
# Then uncomment the reference in kustomization.yaml.
```

Then commit `registry-sealed-secret.yaml` to the repo.

> **Tip:** Create a dedicated Forgejo access token with `read:package` scope
> rather than using your account password.
