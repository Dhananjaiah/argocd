# Simple Nginx Example

A basic Kubernetes deployment for learning Argo CD fundamentals.

## Contents

- `deployment.yaml` - Nginx deployment with 2 replicas
- `service.yaml` - ClusterIP service exposing port 80
- `ingress.yaml` - (Optional) Ingress for external access

## Usage with Argo CD

### Via CLI

```bash
argocd app create simple-nginx \
  --repo https://github.com/YOUR_USERNAME/argocd.git \
  --path examples/simple-nginx \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated
```

### Via UI

1. Click "+ NEW APP"
2. Fill in:
   - Application Name: `simple-nginx`
   - Project: `default`
   - Sync Policy: `Automatic`
   - Repository URL: Your repo URL
   - Path: `examples/simple-nginx`
   - Cluster URL: `https://kubernetes.default.svc`
   - Namespace: `default`
3. Click "CREATE"

### Via YAML Manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: simple-nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/YOUR_USERNAME/argocd.git
    targetRevision: HEAD
    path: examples/simple-nginx
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Testing

```bash
# Check deployment
kubectl get deployment nginx -o wide

# Check pods
kubectl get pods -l app=nginx

# Check service
kubectl get svc nginx

# Port forward to test
kubectl port-forward svc/nginx 8080:80

# Access in browser
open http://localhost:8080
```

## Cleanup

```bash
argocd app delete simple-nginx --yes
```

Or:

```bash
kubectl delete -f .
```
