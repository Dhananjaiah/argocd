# Lecture 4.2: Create Your First Argo CD Application

## Introduction

Now that you have a Git repository with Kubernetes manifests, it's time to create your first Argo CD Application. This Application resource tells Argo CD what to deploy and where.

## Understanding the Application Resource

An Argo CD Application is a Custom Resource Definition (CRD) that defines:
- **Source**: Where the manifests are (Git repo + path)
- **Destination**: Where to deploy (cluster + namespace)
- **Sync Policy**: How to sync (manual or automatic)

## Creating an Application

### Method 1: Using the UI (Easiest)

1. **Access Argo CD UI**:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```
   Open https://localhost:8080

2. **Click "NEW APP"** button

3. **Fill in Application Details**:
   
   **General**:
   - **Application Name**: `nginx-demo`
   - **Project**: `default`
   - **Sync Policy**: `Manual`

   **Source**:
   - **Repository URL**: `https://github.com/<your-username>/argocd-demo-app`
   - **Revision**: `HEAD` (or `main`)
   - **Path**: `.` (root directory)

   **Destination**:
   - **Cluster URL**: `https://kubernetes.default.svc`
   - **Namespace**: `default`

4. **Click "CREATE"**

### Method 2: Using the CLI

```bash
argocd app create nginx-demo \
  --repo https://github.com/<your-username>/argocd-demo-app \
  --path . \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Expected output:
# application 'nginx-demo' created
```

### Method 3: Using kubectl (Declarative)

Create `application.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-demo
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/<your-username>/argocd-demo-app
    targetRevision: HEAD
    path: .
  
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
```

Apply it:

```bash
kubectl apply -f application.yaml -n argocd

# Expected output:
# application.argoproj.io/nginx-demo created
```

## Verifying the Application

### Via UI

1. Go to Applications page
2. You should see `nginx-demo` application
3. Status will show:
   - **Sync Status**: `OutOfSync` (not yet deployed)
   - **Health Status**: `Missing` (resources don't exist)

### Via CLI

```bash
# List applications
argocd app list

# Output:
# NAME        CLUSTER                         NAMESPACE  PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS
# nginx-demo  https://kubernetes.default.svc  default    default  OutOfSync  Missing  <none>      <none>

# Get detailed info
argocd app get nginx-demo
```

## Understanding Application Status

### Sync Status

- **OutOfSync**: Resources in Git differ from cluster (expected initially)
- **Synced**: Git and cluster match
- **Unknown**: Cannot determine status

### Health Status

- **Missing**: Resources don't exist yet (expected initially)
- **Progressing**: Resources being created
- **Healthy**: Resources running correctly
- **Degraded**: Resources have problems

## Key Takeaways

✅ **Application created** successfully  
✅ **OutOfSync status** is expected (haven't synced yet)  
✅ **Three methods** to create apps (UI, CLI, kubectl)  
✅ **Ready to sync** in next lecture

## Next Steps

Continue to [Lecture 4.3: Manual Sync](./lecture-4.3.md) to deploy your application!
