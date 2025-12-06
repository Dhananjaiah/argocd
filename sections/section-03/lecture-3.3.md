# Lecture 3.3: Install Argo CD

## Introduction

Now that you have a running Kubernetes cluster, it's time to install Argo CD! In this lecture, we'll install Argo CD into your cluster and verify the installation is working correctly.

## Installation Methods

There are several ways to install Argo CD:

1. **kubectl apply** - Quick and simple (we'll use this)
2. **Helm chart** - For customized installations
3. **Operator** - For managing multiple Argo CD instances
4. **Kustomize** - For environment-specific configs

For this course, we'll use the **kubectl apply** method as it's the simplest and most common.

## Installation Options

### Standard Installation (Recommended)

Includes all components with default settings.

### High Availability (HA) Installation

For production use with multiple replicas.

### Core Installation

Minimal installation without UI (API/CLI only).

**We'll use the Standard installation** for this course.

## Installing Argo CD

### Step 1: Create Namespace

```bash
# Create argocd namespace
kubectl create namespace argocd

# Verify
kubectl get namespace argocd

# Expected output:
# NAME     STATUS   AGE
# argocd   Active   5s
```

### Step 2: Install Argo CD

```bash
# Install Argo CD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# This will create multiple resources:
# - CustomResourceDefinitions (CRDs)
# - ServiceAccounts
# - ConfigMaps
# - Secrets
# - Services
# - Deployments
# - StatefulSets
```

Expected output:
```
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-server created
...
deployment.apps/argocd-server created
deployment.apps/argocd-repo-server created
statefulset.apps/argocd-application-controller created
...
```

### Step 3: Wait for Pods to be Ready

```bash
# Watch pods until all are Running
kubectl get pods -n argocd --watch

# Or check status
kubectl get pods -n argocd

# Expected output (all Running):
# NAME                                  READY   STATUS    RESTARTS   AGE
# argocd-application-controller-0       1/1     Running   0          2m
# argocd-applicationset-controller-...  1/1     Running   0          2m
# argocd-dex-server-...                 1/1     Running   0          2m
# argocd-notifications-controller-...   1/1     Running   0          2m
# argocd-redis-...                      1/1     Running   0          2m
# argocd-repo-server-...                1/1     Running   0          2m
# argocd-server-...                     1/1     Running   0          2m
```

This may take 2-5 minutes depending on your internet speed.

### Step 4: Verify Installation

```bash
# Check all argocd resources
kubectl get all -n argocd

# Check CRDs
kubectl get crd | grep argoproj

# Expected output:
# applications.argoproj.io
# applicationsets.argoproj.io
# appprojects.argoproj.io
```

## Understanding Installed Components

### Core Components

1. **argocd-server** - API server and web UI
2. **argocd-repo-server** - Manages Git repositories
3. **argocd-application-controller** - Monitors applications
4. **argocd-redis** - Caching layer

### Supporting Components

5. **argocd-dex-server** - SSO and authentication
6. **argocd-notifications-controller** - Notifications
7. **argocd-applicationset-controller** - ApplicationSet management

### Services

```bash
# List services
kubectl get svc -n argocd

# Output:
# NAME                           TYPE        CLUSTER-IP      PORT(S)
# argocd-applicationset-controller ClusterIP   10.96.x.x      7000,8080
# argocd-dex-server              ClusterIP   10.96.x.x      5556,5557,5558
# argocd-metrics                 ClusterIP   10.96.x.x      8082
# argocd-notifications-controller ClusterIP   10.96.x.x      9001
# argocd-redis                   ClusterIP   10.96.x.x      6379
# argocd-repo-server             ClusterIP   10.96.x.x      8081,8084
# argocd-server                  ClusterIP   10.96.x.x      80,443
# argocd-server-metrics          ClusterIP   10.96.x.x      8083
```

## Accessing Argo CD

The Argo CD API server is exposed as a ClusterIP service by default. We need to access it from outside the cluster.

### Option 1: Port Forwarding (Simplest)

```bash
# Forward port 8080 to argocd-server service
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Keep this terminal open
# Access UI at: https://localhost:8080
```

**Pros**: Simple, no cluster changes  
**Cons**: Only works while command runs, one user at a time

### Option 2: NodePort (Kind/Minikube)

```bash
# Patch service to NodePort
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

# Get the NodePort
kubectl get svc argocd-server -n argocd

# Output shows NodePort (e.g., 30080)
# NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)
# argocd-server   NodePort   10.96.x.x     <none>        80:30080/TCP,443:30443/TCP

# For Kind: Access at https://localhost:30443
# For Minikube: Get IP with `minikube ip` then https://<minikube-ip>:30443
```

### Option 3: LoadBalancer (Cloud)

```bash
# Change service type to LoadBalancer
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Get external IP (may take a few minutes)
kubectl get svc argocd-server -n argocd

# For Minikube, run in separate terminal:
minikube tunnel

# Then access at the EXTERNAL-IP shown
```

### Option 4: Ingress (Production)

For production, use an Ingress controller. We'll cover this in later sections.

## Getting the Initial Admin Password

Argo CD generates an initial admin password stored in a secret.

```bash
# Get the password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# Output (example):
# x7gF9kLm2pQr8sVt

# Or on some systems (macOS):
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode; echo
```

**Save this password!** You'll need it to log in.

## Accessing the UI

### Using Port Forward

```bash
# Terminal 1: Keep port forward running
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Open browser to: https://localhost:8080
# Username: admin
# Password: (from previous step)
```

**Note**: Your browser will show a security warning (self-signed certificate). Click "Advanced" and "Proceed" to continue.

### First Login

1. Navigate to https://localhost:8080
2. Accept the certificate warning
3. Login with:
   - **Username**: `admin`
   - **Password**: (from secret)
4. You should see the Argo CD UI!

## Alternative Installation Methods

### High Availability Installation

For production with multiple replicas:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml
```

### Core Installation

Minimal installation without UI:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml
```

### Using Helm

```bash
# Add Argo CD Helm repo
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Install with Helm
helm install argocd argo/argo-cd --namespace argocd --create-namespace

# Or with custom values
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  --set server.service.type=LoadBalancer
```

## Verification Checklist

✅ All pods are Running:
```bash
kubectl get pods -n argocd
```

✅ Services are created:
```bash
kubectl get svc -n argocd
```

✅ Can access UI:
```bash
# Port forward running
# Browser shows Argo CD login page
```

✅ Can log in with admin credentials:
```bash
# Username: admin
# Password: (from secret)
```

## Troubleshooting

### Pods not starting

```bash
# Check pod status
kubectl get pods -n argocd

# Describe pod for errors
kubectl describe pod <pod-name> -n argocd

# Check logs
kubectl logs <pod-name> -n argocd
```

Common issues:
- Insufficient resources (increase Docker/Minikube memory)
- Image pull errors (check internet connection)
- Node not ready (restart cluster)

### Can't access UI

```bash
# Verify port forward is running
netstat -an | grep 8080

# Check service exists
kubectl get svc argocd-server -n argocd

# Try different port
kubectl port-forward svc/argocd-server -n argocd 9090:443
```

### Certificate errors in browser

This is expected with self-signed certificates. Click "Advanced" → "Proceed" or "Accept Risk".

To fix properly (optional):
```bash
# Skip TLS verification (not for production!)
kubectl patch cm argocd-cmd-params-cm -n argocd \
  --type merge \
  -p '{"data":{"server.insecure":"true"}}'

# Restart server
kubectl rollout restart deployment argocd-server -n argocd
```

### Can't retrieve initial password

```bash
# Check if secret exists
kubectl get secret argocd-initial-admin-secret -n argocd

# If missing, reset admin password:
kubectl -n argocd patch secret argocd-secret \
  -p '{"stringData": {"admin.password": "'$(htpasswd -bnBC 10 "" <new-password> | tr -d ':\n')'"}}'
```

## Best Practices

✅ **Change admin password** after first login  
✅ **Use port-forward for learning** (simple and secure)  
✅ **Use Ingress for production** (with proper TLS)  
✅ **Keep Argo CD updated** (check for security updates)  
✅ **Backup argocd namespace** regularly

## Key Takeaways

✅ **Installation is simple** with kubectl apply  
✅ **Multiple access methods** available  
✅ **Port-forward works best** for local learning  
✅ **Initial password** is in a secret  
✅ **All components running** means ready to use

## Common Questions

**Q: How long does installation take?**  
A: 2-5 minutes depending on internet speed for image pulls.

**Q: Can I install Argo CD in a different namespace?**  
A: Yes, but `argocd` namespace is standard. Update all commands accordingly.

**Q: Should I use HA installation for learning?**  
A: No, standard installation is sufficient. HA is for production.

**Q: Can I access Argo CD from a different machine?**  
A: With port-forward, no. Use NodePort, LoadBalancer, or Ingress instead.

**Q: Is the initial password secure?**  
A: It's randomly generated and secure. Change it after first login.

## Next Steps

Now that Argo CD is installed and accessible, continue to [Lecture 3.4: Access UI and CLI Login](./lecture-3.4.md) to learn how to interact with Argo CD.
