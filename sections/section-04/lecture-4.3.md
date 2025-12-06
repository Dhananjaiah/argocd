# Lecture 4.3: Manual Sync

## Introduction

Your Application is created but not yet deployed. In this lecture, you'll perform your first sync operation to deploy the application to your cluster.

## What is Sync?

**Sync** is the process of applying the desired state from Git to your Kubernetes cluster. It's the core GitOps operation that makes your cluster match your Git repository.

## Performing Manual Sync

### Via UI

1. **Open Application**: Click on `nginx-demo`

2. **Review Resources**: You'll see the resources that will be created:
   - Deployment: nginx-demo
   - Service: nginx-demo
   - ConfigMap: nginx-demo-config

3. **Click SYNC Button**: Top right

4. **Review Sync Options**:
   - **Prune**: ❌ Off (don't delete extra resources)
   - **Dry Run**: ❌ Off (actually apply changes)
   - **Apply Only**: Choose "All resources" or select specific ones

5. **Click SYNCHRONIZE**

6. **Watch Progress**: Resources will appear in the tree view

### Via CLI

```bash
# Sync the application
argocd app sync nginx-demo

# Expected output:
# TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
# 2024-01-15T10:00:00+00:00            Service     default           nginx-demo  OutOfSync  Missing
# 2024-01-15T10:00:00+00:00   apps  Deployment     default           nginx-demo  OutOfSync  Missing
# 2024-01-15T10:00:00+00:00        ConfigMap     default  nginx-demo-config  OutOfSync  Missing
# ...
# service/nginx-demo created
# deployment.apps/nginx-demo created
# configmap/nginx-demo-config created
```

## Monitoring Sync Progress

### Via UI

Watch the application details page:
1. **Sync Status**: Changes from OutOfSync → Synced
2. **Health Status**: Missing → Progressing → Healthy
3. **Resource Tree**: Shows resources appearing and becoming healthy

### Via CLI

```bash
# Watch sync progress
argocd app get nginx-demo --refresh

# Or watch continuously
watch argocd app get nginx-demo

# Check specific resources
kubectl get pods -n default -l app=nginx-demo
```

## Verifying Deployment

### Check Pods

```bash
# Get pods
kubectl get pods -n default -l app=nginx-demo

# Expected output:
# NAME                          READY   STATUS    RESTARTS   AGE
# nginx-demo-7d8b9c-abc         1/1     Running   0          1m
# nginx-demo-7d8b9c-def         1/1     Running   0          1m
```

### Check Service

```bash
# Get service
kubectl get svc nginx-demo -n default

# Expected output:
# NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
# nginx-demo   ClusterIP   10.96.123.45   <none>        80/TCP    1m
```

### Test Application

```bash
# Port forward to test
kubectl port-forward svc/nginx-demo -n default 8081:80

# In another terminal or browser:
curl http://localhost:8081

# Should see nginx welcome page!
```

## Understanding Sync Options

### Prune

**What it does**: Delete resources that exist in cluster but not in Git

```bash
# Sync with prune
argocd app sync nginx-demo --prune
```

**Use with caution!** Can delete resources accidentally.

### Dry Run

**What it does**: Show what would change without actually applying

```bash
# Dry run
argocd app sync nginx-demo --dry-run

# Shows what would be created/updated/deleted
# But doesn't actually do it
```

### Selective Sync

**What it does**: Sync only specific resources

```bash
# Sync only the deployment
argocd app sync nginx-demo --resource apps:Deployment:nginx-demo
```

## Sync Waves

Resources can be synced in waves using annotations:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "0"  # Default wave
```

Lower numbers sync first. We'll cover this in advanced sections.

## Post-Sync Verification

### Application Status

```bash
# Get application status
argocd app get nginx-demo

# Should show:
# Sync Status:      Synced
# Health Status:    Healthy
```

### Resource Status

```bash
# Check all resources
kubectl get all -n default -l app=nginx-demo

# Output:
# NAME                              READY   STATUS    RESTARTS   AGE
# pod/nginx-demo-7d8b9c-abc         1/1     Running   0          2m
# pod/nginx-demo-7d8b9c-def         1/1     Running   0          2m
#
# NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
# service/nginx-demo   ClusterIP   10.96.123.45   <none>        80/TCP    2m
#
# NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/nginx-demo   2/2     2            2           2m
```

## Making Changes

### Update Git Repository

```bash
# Edit deployment.yaml - change replicas to 3
cd argocd-demo-app
sed -i 's/replicas: 2/replicas: 3/' deployment.yaml

# Commit and push
git add deployment.yaml
git commit -m "Scale to 3 replicas"
git push
```

### Observe OutOfSync

```bash
# Refresh Argo CD (it checks Git every 3 minutes)
argocd app get nginx-demo --refresh

# Status will show OutOfSync
```

### Sync Again

```bash
# Sync to apply changes
argocd app sync nginx-demo

# Verify new replica
kubectl get pods -n default -l app=nginx-demo
# Should now show 3 pods!
```

## Troubleshooting

### Sync fails immediately

**Check**:
```bash
# View sync operation details
argocd app get nginx-demo

# Check events
kubectl get events -n default --sort-by='.lastTimestamp'
```

### Resources not creating

**Check**:
```bash
# Describe the resource
kubectl describe deployment nginx-demo -n default

# Check pod logs if applicable
kubectl logs -n default -l app=nginx-demo
```

### Permission errors

**Error**: `User "system:serviceaccount:argocd:argocd-application-controller" cannot create deployments`

**Solution**: Argo CD needs RBAC permissions (usually set up by default for default cluster).

## Key Takeaways

✅ **Manual sync** gives you control  
✅ **Review before syncing** is good practice  
✅ **Both UI and CLI** work equally well  
✅ **Synced + Healthy** means success  
✅ **Git changes** require re-sync

## Next Steps

Continue to [Lecture 4.4: Debugging Common First-Time Issues](./lecture-4.4.md) to learn troubleshooting!
