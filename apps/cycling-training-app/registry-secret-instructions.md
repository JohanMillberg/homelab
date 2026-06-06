# Registry pull secret setup for cycling-training-app

Before Flux reconciles this app, you must create the registry pull
secret on the server so the cluster can pull images from forgejo.lan.

Run on the homelab server (192.168.1.205):

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl create secret docker-registry forgejo-registry-secret \
    --docker-server=forgejo.lan \
    --docker-username=kvarnberg \
    --docker-password=<your-forgejo-token> \
    --namespace=cycling-training-app \
    --dry-run=client -o yaml \
  | kubeseal --kubeconfig /home/johan/.kube/config -o yaml"
```

Write the output to `apps/cycling-training-app/registry-sealed-secret.yaml`.

Then create the app SealedSecret for env vars:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config \
  kubectl create secret generic cycling-training-app-secrets \
    --namespace=cycling-training-app \
    --from-literal=SECRET_KEY='<random-64-char-hex>' \
    --from-literal=STRAVA_CLIENT_ID='<your-strava-client-id>' \
    --from-literal=STRAVA_CLIENT_SECRET='<your-strava-secret>' \
    --from-literal=LLM_API_KEY='<your-opencode-api-key>' \
    --from-literal=INTERVALS_API_KEY='<your-intervals-api-key>' \
    --from-literal=INTERVALS_ATHLETE_ID='<your-intervals-athlete-id>' \
    --dry-run=client -o yaml \
  | kubeseal --kubeconfig /home/johan/.kube/config -o yaml"
```

Write output to `apps/cycling-training-app/sealed-secret.yaml`.

## First-time setup

The namespace must exist before kubeseal runs:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl apply -f -" << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: cycling-training-app
EOF
```
