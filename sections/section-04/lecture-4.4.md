# Lecture 4.4: Debugging Common First-Time Issues

## Introduction

Even with perfect setup, you'll encounter issues when deploying applications. This lecture covers common problems and how to debug them effectively.

## Common Issues and Solutions

### 1. Application Stuck in OutOfSync

**Symptom**: Application remains OutOfSync after creating

**Cause**: Argo CD hasn't synced yet (manual sync policy)

**Solution**:
```bash
# Simply sync the application
argocd app sync <app-name>

# Or enable auto-sync
argocd app set <app-name> --sync-policy automated
```

### 2. Pods in CrashLoopBackOff

**Symptom**: Pods keep restarting

**Debug**:
```bash
# Check pod status
kubectl get pods -n <namespace>

# View logs
kubectl logs <pod-name> -n <namespace>

# Describe pod for events
kubectl describe pod <pod-name> -n <namespace>
```

**Common Causes**:
- Application error in container
- Missing environment variables
- Port already in use
- Health check failing

### 3. Image Pull Errors

**Symptom**: Pods stuck in ImagePullBackOff or ErrImagePull

**Debug**:
```bash
kubectl describe pod <pod-name> -n <namespace>

# Look for:
# Failed to pull image "nginx:wrong-tag": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:wrong-tag not found
```

**Solutions**:
```bash
# Fix image name/tag in Git
# Common issues:
# - Typo in image name
# - Non-existent tag
# - Private registry without credentials
# - Wrong registry URL
```

### 4. Service Has No Endpoints

**Symptom**: Service exists but has no endpoints

**Debug**:
```bash
# Check service endpoints
kubectl get endpoints <service-name> -n <namespace>

# Should show pod IPs
# If empty, check selector mismatch
kubectl describe service <service-name> -n <namespace>
kubectl get pods -n <namespace> --show-labels
```

**Cause**: Label selector mismatch between Service and Pods

**Solution**: Ensure Service selector matches Pod labels exactly.

### 5. Permission Denied Errors

**Symptom**: Sync fails with RBAC errors

**Error Message**: `User "system:serviceaccount:argocd:argocd-application-controller" cannot create deployments`

**Solution**:
```bash
# For default cluster, Argo CD has full permissions
# For external clusters, ensure proper RBAC setup

# Check Argo CD service account permissions
kubectl describe clusterrolebinding argocd-application-controller
```

### 6. Repository Connection Failures

**Symptom**: Can't fetch from Git repository

**Error**: `rpc error: code = Unknown desc = authentication required` or `repository not accessible`

**Debug**:
```bash
# Test repository connection
argocd repo get https://github.com/<user>/<repo>

# Add repository if missing
argocd repo add https://github.com/<user>/<repo>

# For private repos, add credentials
argocd repo add https://github.com/<user>/<repo> \
  --username <username> \
  --password <token>
```

### 7. Namespace Doesn't Exist

**Symptom**: Resources fail to create

**Error**: `namespaces "<namespace>" not found`

**Solution**:
```bash
# Create namespace manually
kubectl create namespace <namespace>

# Or add sync option to create it automatically
argocd app set <app-name> --sync-option CreateNamespace=true

# In Application YAML:
spec:
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
```

### 8. Resources Not Updating

**Symptom**: Git changes don't appear in cluster after sync

**Causes**:
- Argo CD cache delay (3 minute default)
- Looking at wrong cluster/namespace
- Wrong branch/revision

**Debug**:
```bash
# Force refresh from Git
argocd app get <app-name> --refresh --hard-refresh

# Check which revision is deployed
argocd app get <app-name> | grep "Target Revision"

# Verify you're looking at correct cluster
kubectl config current-context
```

### 9. Health Check Failing

**Symptom**: Application shows Degraded or Progressing

**Debug**:
```bash
# Check specific resource health
argocd app get <app-name>

# Look at resource conditions
kubectl describe <resource-type> <resource-name> -n <namespace>

# Check pod readiness
kubectl get pods -n <namespace>
```

**Common Causes**:
- Readiness probe failing
- Liveness probe failing
- Insufficient resources
- Application startup slow

### 10. Sync Takes Forever

**Symptom**: Sync operation hangs

**Debug**:
```bash
# Check sync operation status
argocd app get <app-name>

# View detailed sync status
kubectl get application <app-name> -n argocd -o yaml

# Check if resources are being created
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

**Common Causes**:
- Image pull timeout
- Init container not completing
- PersistentVolume provisioning delay
- Webhook admission controller slow

## Debugging Workflow

### Step 1: Check Application Status

```bash
# Get overview
argocd app get <app-name>

# Check for:
# - Sync status (Synced/OutOfSync)
# - Health status (Healthy/Degraded/Progressing)
# - Last sync result
# - Recent events
```

### Step 2: Check Kubernetes Resources

```bash
# Get all resources
kubectl get all -n <namespace> -l app=<app-name>

# Check events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

### Step 3: Examine Specific Resources

```bash
# Describe resource for details
kubectl describe <resource-type> <name> -n <namespace>

# Check logs
kubectl logs <pod-name> -n <namespace>

# For previous crashed pod
kubectl logs <pod-name> -n <namespace> --previous
```

### Step 4: Check Argo CD Logs

```bash
# Application controller logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller

# Repo server logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-repo-server

# API server logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server
```

## Useful Commands Reference

### Argo CD

```bash
# App info
argocd app get <app-name>

# Refresh app
argocd app get <app-name> --refresh

# View diff
argocd app diff <app-name>

# Sync with details
argocd app sync <app-name> --dry-run

# Delete app
argocd app delete <app-name>
```

### Kubectl

```bash
# Get resources
kubectl get <resource> -n <namespace>

# Describe
kubectl describe <resource> <name> -n <namespace>

# Logs
kubectl logs <pod> -n <namespace>

# Events
kubectl get events -n <namespace>

# Execute commands in pod
kubectl exec -it <pod> -n <namespace> -- /bin/sh
```

## Key Takeaways

✅ **Check logs first** - they usually tell you what's wrong  
✅ **Events are helpful** - kubectl events show recent issues  
✅ **Describe resources** - shows detailed status  
✅ **UI shows visual status** - easy to spot problems  
✅ **Patience** - some operations take time

## Common Questions

**Q: Why is my app OutOfSync?**  
A: Either you haven't synced yet, or Git has changes not applied.

**Q: Why are pods not starting?**  
A: Check logs and describe pod. Usually image or config issues.

**Q: How do I reset an application?**  
A: Delete and recreate, or sync with prune.

**Q: Can I roll back a deployment?**  
A: Yes, sync to a previous Git commit.

## Next Steps

Congratulations on completing Section 4! You've deployed your first GitOps application!

Continue to [Section 5: Sync Policies Deep Dive](../section-05/README.md) to learn about automated sync!

Complete **[Lab 2: Your First GitOps Deployment](../../labs/lab-02.md)**

Don't forget the [Section 4 Quiz](../../quizzes/section-04-quiz.md)!
