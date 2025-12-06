# Lab 1: Setting Up Your Argo CD Environment

**Difficulty**: Beginner  
**Duration**: 30 minutes  
**Prerequisites**: Docker installed

## Goal

Set up a local Kubernetes cluster using Kind, install Argo CD, and access both the UI and CLI.

## What You'll Learn

- How to create a local Kubernetes cluster with Kind
- How to install Argo CD using kubectl
- How to access the Argo CD UI
- How to use the Argo CD CLI

## Prerequisites

- Docker installed and running
- Terminal/command line access
- Internet connection for downloading tools

## Step 1: Install Required Tools

### Install kubectl

**macOS**:
```bash
brew install kubectl
```

**Linux**:
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

**Windows** (PowerShell):
```powershell
choco install kubernetes-cli
```

**Verify**:
```bash
kubectl version --client
```

### Install Kind

**macOS**:
```bash
brew install kind
```

**Linux**:
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

**Windows**:
```powershell
choco install kind
```

**Verify**:
```bash
kind version
```

### Install Argo CD CLI

**macOS**:
```bash
brew install argocd
```

**Linux**:
```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd
```

**Windows**:
```powershell
choco install argocd-cli
```

**Verify**:
```bash
argocd version --client
```

## Step 2: Create Kubernetes Cluster

Create a Kind cluster:

```bash
kind create cluster --name argocd-lab
```

**Expected Output**:
```
Creating cluster "argocd-lab" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-argocd-lab"
```

**Verify cluster is running**:
```bash
kubectl cluster-info --context kind-argocd-lab
```

**Check nodes**:
```bash
kubectl get nodes
```

**Expected Output**:
```
NAME                       STATUS   ROLES           AGE   VERSION
argocd-lab-control-plane   Ready    control-plane   1m    v1.27.3
```

## Step 3: Install Argo CD

### Create Argo CD Namespace

```bash
kubectl create namespace argocd
```

### Install Argo CD

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**What this installs**:
- Argo CD API Server
- Argo CD Repository Server
- Argo CD Application Controller
- Argo CD Redis (for caching)
- Argo CD Server UI

### Wait for Argo CD to be Ready

```bash
kubectl wait --for=condition=available --timeout=300s \
  deployment/argocd-server -n argocd
```

**Check all pods are running**:
```bash
kubectl get pods -n argocd
```

**Expected Output** (all pods should be Running):
```
NAME                                  READY   STATUS    RESTARTS   AGE
argocd-application-controller-0       1/1     Running   0          2m
argocd-applicationset-controller-..   1/1     Running   0          2m
argocd-dex-server-...                 1/1     Running   0          2m
argocd-notifications-controller-...   1/1     Running   0          2m
argocd-redis-...                      1/1     Running   0          2m
argocd-repo-server-...                1/1     Running   0          2m
argocd-server-...                     1/1     Running   0          2m
```

## Step 4: Access Argo CD UI

### Port Forward to Argo CD Server

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
```

**Note**: The `&` runs it in the background. You can also open a new terminal.

### Get Admin Password

The initial admin password is auto-generated and stored in a secret:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```

**Save this password!** You'll need it to log in.

**Example output**:
```
xJ8kF9pQnR2mL4vB
```

### Access the UI

1. Open browser to: https://localhost:8080
2. Accept the self-signed certificate warning
3. Username: `admin`
4. Password: (the password from previous command)

**You should see the Argo CD dashboard!**

## Step 5: Login with Argo CD CLI

```bash
argocd login localhost:8080 --insecure
```

**Enter credentials**:
- Username: `admin`
- Password: (your admin password)

**Expected Output**:
```
'admin:login' logged in successfully
Context 'localhost:8080' updated
```

### Update Admin Password (Recommended)

```bash
argocd account update-password
```

Enter:
- Current password: (auto-generated password)
- New password: (your chosen password)
- Confirm: (your chosen password)

## Verification Commands

### 1. Check Argo CD Version

```bash
argocd version
```

**Expected Output**:
```
argocd: v2.9.0+...
  BuildDate: ...
argocd-server: v2.9.0+...
  BuildDate: ...
```

### 2. Check Cluster Connection

```bash
argocd cluster list
```

**Expected Output**:
```
SERVER                          NAME        VERSION  STATUS   MESSAGE
https://kubernetes.default.svc  in-cluster  1.27     Successful
```

### 3. Check Application List (should be empty)

```bash
argocd app list
```

**Expected Output**:
```
NAME  CLUSTER  NAMESPACE  PROJECT  STATUS  HEALTH  SYNCPOLICY  CONDITIONS
```

(Empty list is expected - we haven't created any applications yet)

### 4. Verify kubectl Context

```bash
kubectl config current-context
```

**Expected Output**:
```
kind-argocd-lab
```

## Troubleshooting

### Issue: Pods Not Starting

**Check pod status**:
```bash
kubectl get pods -n argocd
kubectl describe pod <pod-name> -n argocd
```

**Common causes**:
- Insufficient resources (Docker needs 4GB+ RAM)
- Image pull issues (check internet connection)

**Solution**: Delete and recreate cluster:
```bash
kind delete cluster --name argocd-lab
kind create cluster --name argocd-lab
# Re-run installation steps
```

### Issue: Cannot Access UI

**Check port-forward is running**:
```bash
ps aux | grep port-forward
```

**If not running, restart**:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**Check service exists**:
```bash
kubectl get svc -n argocd argocd-server
```

### Issue: Certificate Errors in Browser

This is normal for local development. Click "Advanced" → "Proceed to localhost (unsafe)" or similar.

**Alternatively, use --insecure flag**:
```bash
argocd login localhost:8080 --insecure
```

### Issue: Admin Password Not Working

**Reset the password**:
```bash
# Get new password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo

# Or delete the secret to generate a new one
kubectl delete secret argocd-initial-admin-secret -n argocd
kubectl rollout restart deployment argocd-server -n argocd
```

### Issue: Kind Cluster Creation Fails

**Check Docker is running**:
```bash
docker ps
```

**Check Docker resources**:
- Ensure Docker Desktop has at least 4GB RAM allocated
- Check available disk space

**Use different cluster name**:
```bash
kind create cluster --name mylab
```

## Cleanup

When you're done with the lab (DON'T do this yet if continuing to next labs):

```bash
# Delete the Kind cluster
kind delete cluster --name argocd-lab

# Verify deletion
kind get clusters
```

**Note**: This deletes everything including Argo CD installation.

## What You've Accomplished

✅ Installed kubectl, Kind, and Argo CD CLI  
✅ Created a local Kubernetes cluster  
✅ Installed Argo CD in the cluster  
✅ Accessed Argo CD UI  
✅ Logged in via Argo CD CLI  
✅ Verified installation  

## Next Steps

You now have a working Argo CD environment! 

Continue to:
- **[Lab 2: Your First GitOps Deployment](./lab-02.md)** - Deploy your first application with Argo CD
- **[Section 3: Lab Setup](../sections/section-03/README.md)** - Learn more about the tools

## Additional Resources

- [Argo CD Documentation](https://argo-cd.readthedocs.io/)
- [Kind Documentation](https://kind.sigs.k8s.io/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Quick Reference

```bash
# Start port-forward (if not running)
kubectl port-forward svc/argocd-server -n argocd 8080:443 &

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo

# CLI login
argocd login localhost:8080 --insecure

# List apps
argocd app list

# Check cluster
kubectl cluster-info
kubectl get nodes
kubectl get pods -n argocd
```

---

**Great job completing Lab 1!** Your environment is ready for GitOps adventures.
