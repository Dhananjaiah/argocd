# Common Issues and Troubleshooting Guide

Solutions to common problems when learning and using Argo CD.

## Installation Issues

### Issue: Kind cluster won't start

**Symptoms**:
```
ERROR: failed to create cluster: ...
```

**Solutions**:
1. Check Docker is running:
   ```bash
   docker ps
   ```

2. Check Docker resources (need 4GB+ RAM):
   - Docker Desktop → Settings → Resources

3. Delete and recreate:
   ```bash
   kind delete cluster --name argocd-lab
   kind create cluster --name argocd-lab
   ```

### Issue: Argo CD pods not starting

**Symptoms**:
```
argocd-server pod stuck in Pending or CrashLoopBackOff
```

**Solutions**:
1. Check pod status:
   ```bash
   kubectl get pods -n argocd
   kubectl describe pod <pod-name> -n argocd
   ```

2. Check events:
   ```bash
   kubectl get events -n argocd --sort-by='.lastTimestamp'
   ```

3. Common causes:
   - Insufficient resources
   - Image pull errors
   - Node not ready

4. Fix: Increase Docker resources or recreate cluster

### Issue: Can't access Argo CD UI

**Symptoms**:
```
localhost:8080 connection refused
```

**Solutions**:
1. Check port-forward is running:
   ```bash
   ps aux | grep port-forward
   ```

2. Restart port-forward:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443 &
   ```

3. Check service exists:
   ```bash
   kubectl get svc -n argocd argocd-server
   ```

4. Accept certificate warning in browser

## Authentication Issues

### Issue: Can't login to Argo CD

**Symptoms**:
```
FATA[0000] Failed to establish connection to localhost:8080
```

**Solutions**:
1. Get current password:
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret \
     -o jsonpath="{.data.password}" | base64 -d; echo
   ```

2. Try with `--insecure` flag:
   ```bash
   argocd login localhost:8080 --insecure
   ```

3. Reset password:
   ```bash
   kubectl delete secret argocd-initial-admin-secret -n argocd
   kubectl rollout restart deployment argocd-server -n argocd
   ```

## Application Issues

### Issue: Application stuck OutOfSync

**Symptoms**:
```
Sync Status: OutOfSync from main (abc123)
```

**Solutions**:
1. Check if auto-sync is enabled:
   ```bash
   argocd app get <app-name> | grep "Sync Policy"
   ```

2. Manual sync:
   ```bash
   argocd app sync <app-name>
   ```

3. Check for errors:
   ```bash
   argocd app get <app-name>
   ```

4. Hard refresh:
   ```bash
   argocd app get <app-name> --hard-refresh
   ```

### Issue: Application shows Degraded health

**Symptoms**:
```
Health Status: Degraded
```

**Solutions**:
1. Check pod status:
   ```bash
   kubectl get pods -n <namespace>
   ```

2. Describe pod:
   ```bash
   kubectl describe pod <pod-name> -n <namespace>
   ```

3. Check logs:
   ```bash
   kubectl logs <pod-name> -n <namespace>
   ```

4. Common causes:
   - Image doesn't exist
   - Resource limits too low
   - Missing dependencies
   - Configuration errors

### Issue: Resources not being created

**Symptoms**:
```
Sync Status: Synced
Health Status: Missing
```

**Solutions**:
1. Check for errors in Argo CD:
   ```bash
   argocd app get <app-name>
   ```

2. Check events:
   ```bash
   kubectl get events -n <namespace> --sort-by='.lastTimestamp'
   ```

3. Verify namespace exists:
   ```bash
   kubectl get namespace <namespace>
   ```

4. Check RBAC permissions:
   ```bash
   kubectl auth can-i create deployment --as=system:serviceaccount:argocd:argocd-application-controller -n <namespace>
   ```

## Git Repository Issues

### Issue: Can't access Git repository

**Symptoms**:
```
rpc error: code = Unknown desc = authentication required
```

**Solutions**:
1. For public repos, use HTTPS:
   ```
   https://github.com/user/repo.git
   ```

2. For private repos, add credentials:
   ```bash
   argocd repo add https://github.com/user/repo.git \
     --username <user> \
     --password <token>
   ```

3. For SSH, add SSH key:
   ```bash
   argocd repo add git@github.com:user/repo.git \
     --ssh-private-key-path ~/.ssh/id_rsa
   ```

### Issue: Argo CD not detecting Git changes

**Symptoms**:
```
Changes pushed to Git but application not updating
```

**Solutions**:
1. Check sync policy:
   ```bash
   argocd app get <app-name> | grep "Sync Policy"
   ```

2. Manual refresh:
   ```bash
   argocd app get <app-name> --refresh
   ```

3. Check webhook (if configured):
   ```bash
   # Verify webhook in Git provider settings
   ```

4. Wait for poll interval (default 3 minutes)

## Sync Issues

### Issue: Self-heal not working

**Symptoms**:
```
Manual changes not being reverted
```

**Solutions**:
1. Check if enabled:
   ```bash
   argocd app get <app-name> | grep -A 3 "Sync Policy"
   ```

2. Enable self-heal:
   ```bash
   argocd app set <app-name> --self-heal
   ```

3. Wait for sync interval (default 3 minutes)

4. Check application controller logs:
   ```bash
   kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller --tail=50
   ```

### Issue: Prune deleting wrong resources

**Symptoms**:
```
Resources being deleted unexpectedly
```

**Solutions**:
1. Check resource labels:
   ```bash
   kubectl get <resource> -n <namespace> --show-labels
   ```

2. Disable prune temporarily:
   ```bash
   argocd app set <app-name> --sync-option Prune=false
   ```

3. Add annotation to exclude from pruning:
   ```yaml
   metadata:
     annotations:
       argocd.argoproj.io/sync-options: Prune=false
   ```

## Kustomize Issues

### Issue: Kustomize build fails

**Symptoms**:
```
error: accumulating resources: ...
```

**Solutions**:
1. Test locally:
   ```bash
   kustomize build overlays/<env>
   ```

2. Check file paths in kustomization.yaml

3. Verify YAML syntax:
   ```bash
   yamllint kustomization.yaml
   ```

4. Check base path is correct

### Issue: Wrong overlay being used

**Symptoms**:
```
Application using dev config in prod
```

**Solutions**:
1. Check application path:
   ```bash
   argocd app get <app-name> | grep Path
   ```

2. Update path:
   ```bash
   argocd app set <app-name> --path overlays/prod
   ```

## Performance Issues

### Issue: Argo CD UI slow or unresponsive

**Solutions**:
1. Check resource usage:
   ```bash
   kubectl top pods -n argocd
   ```

2. Increase resources:
   ```yaml
   # Edit deployment
   kubectl edit deployment argocd-server -n argocd
   # Increase CPU/memory requests and limits
   ```

3. Reduce sync frequency:
   ```bash
   # Edit ConfigMap
   kubectl edit configmap argocd-cm -n argocd
   # Add: timeout.reconciliation: 10m
   ```

### Issue: Too many applications causing slowness

**Solutions**:
1. Use projects to organize apps

2. Increase controller resources

3. Use multiple Argo CD instances

4. Enable sharding for large deployments

## Debugging Commands

### Get detailed information

```bash
# Application details
argocd app get <app-name>

# Application manifests
argocd app manifests <app-name>

# Application diff
argocd app diff <app-name>

# Application history
argocd app history <app-name>

# Hard refresh (clear cache)
argocd app get <app-name> --hard-refresh
```

### Check Kubernetes resources

```bash
# All resources in namespace
kubectl get all -n <namespace>

# Specific resource
kubectl get <resource> -n <namespace>
kubectl describe <resource> <name> -n <namespace>

# Events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'

# Logs
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous  # Previous container
```

### Check Argo CD components

```bash
# Argo CD pods
kubectl get pods -n argocd

# Application controller logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller --tail=100

# Server logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server --tail=100

# Repo server logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-repo-server --tail=100
```

## Best Practices to Avoid Issues

1. **Start Simple**: Begin with basic YAML before Kustomize/Helm
2. **Test Locally**: Use `kubectl apply --dry-run=client -f manifest.yaml`
3. **Check Syntax**: Validate YAML before committing
4. **Use Version Control**: Commit everything to Git
5. **Monitor Logs**: Watch Argo CD logs when troubleshooting
6. **Read Error Messages**: Argo CD provides detailed error info
7. **Use Labels**: Properly label resources for tracking
8. **Backup**: Keep backups of Argo CD configuration

## Getting More Help

1. **Check logs** first (Argo CD and Kubernetes)
2. **Search** [Argo CD GitHub issues](https://github.com/argoproj/argo-cd/issues)
3. **Ask** on [Argo CD Slack](https://argoproj.github.io/community/join-slack/)
4. **Review** [Official Documentation](https://argo-cd.readthedocs.io/)

---

**Remember**: Most issues are configuration errors. Double-check your YAML syntax and paths!
