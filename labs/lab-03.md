# Lab 3: Automated Sync and Self-Heal

**Difficulty**: Beginner  
**Duration**: 25 minutes  
**Prerequisites**: Lab 2 completed

## Goal

Configure automatic synchronization and self-healing in Argo CD to achieve true GitOps automation.

## What You'll Learn

- How to enable automated sync
- What prune and self-heal do
- How drift detection works
- When to use automated vs manual sync

## Step 1: Create Application with Auto-Sync

Let's create a new application with automated sync enabled.

```bash
argocd app create nginx-auto \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated
```

**Notice the `--sync-policy automated` flag!**

### Check Application Status

```bash
argocd app get nginx-auto
```

**Output shows**:
```
Sync Policy:        Automated
```

**The application will sync automatically** - no manual sync needed!

## Step 2: Enable Prune

Prune automatically deletes resources that are removed from Git.

```bash
argocd app set nginx-auto --sync-option Prune=true
```

**Or create with prune enabled**:
```bash
argocd app create nginx-prune \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --auto-prune
```

### What Prune Does

```yaml
# Scenario:
# 1. You have 3 manifests in Git: deployment.yaml, service.yaml, configmap.yaml
# 2. All are deployed to cluster
# 3. You remove configmap.yaml from Git
# 4. Without prune: ConfigMap stays in cluster (drift)
# 5. With prune: ConfigMap is automatically deleted
```

## Step 3: Enable Self-Heal

Self-heal automatically reverts manual changes made directly to the cluster.

```bash
argocd app set nginx-auto --self-heal
```

**Or create with self-heal enabled**:
```bash
argocd app create nginx-selfheal \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --self-heal \
  --auto-prune
```

### What Self-Heal Does

```yaml
# Scenario:
# 1. Git says: replicas: 2
# 2. Cluster has: replicas: 2
# 3. Someone runs: kubectl scale deployment/myapp --replicas=5
# 4. Without self-heal: Cluster stays at 5 replicas (drift)
# 5. With self-heal: Argo CD detects change and reverts to 2 replicas
```

## Step 4: Test Self-Heal

Let's see self-heal in action!

### Deploy Application

```bash
argocd app create test-selfheal \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --self-heal
```

### Wait for Sync

```bash
argocd app wait test-selfheal --health
```

### Check Current Replicas

```bash
kubectl get deployment guestbook-ui -o jsonpath='{.spec.replicas}'
```

Should show: `1`

### Make Manual Change

```bash
kubectl scale deployment guestbook-ui --replicas=5
```

### Watch What Happens

```bash
# Watch the deployment
watch kubectl get deployment guestbook-ui

# Watch Argo CD events
argocd app get test-selfheal --refresh
```

**Within 3-5 minutes, Argo CD will detect the drift and revert to 1 replica!**

You can also see this in the UI - the app will briefly show "OutOfSync" then auto-sync back.

## Step 5: Test Prune

Let's test the prune functionality.

### Create Application with Prune

```bash
argocd app create test-prune \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --auto-prune
```

### Add Extra Resource Manually

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: extra-config
  namespace: default
  labels:
    app.kubernetes.io/instance: test-prune
data:
  key: value
EOF
```

### Label it so Argo CD Tracks It

```bash
kubectl label configmap extra-config -n default \
  app.kubernetes.io/instance=test-prune
```

### Trigger Sync

```bash
argocd app sync test-prune
```

**The ConfigMap will be deleted** because it's not in Git!

### Verify

```bash
kubectl get configmap extra-config -n default
```

Should show: `Error from server (NotFound)`

## Step 6: Create via YAML Manifest

You can also create applications using YAML manifests. This is preferred for GitOps!

```bash
cat <<'EOF' > application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    
    syncOptions:
      - CreateNamespace=true
    
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
EOF
```

### Apply It

```bash
kubectl apply -f application.yaml
```

### Verify

```bash
argocd app get my-app
```

## Step 7: Sync Options Deep Dive

### Available Sync Options

```yaml
syncOptions:
  - CreateNamespace=true       # Create namespace if not exists
  - PrunePropagationPolicy=foreground  # How to delete resources
  - PruneLast=true            # Prune after other resources are healthy
  - Validate=false            # Skip kubectl validation
  - ApplyOutOfSyncOnly=true   # Only sync resources that are out-of-sync
  - RespectIgnoreDifferences=true  # Honor ignoreDifferences
```

### Example with Multiple Options

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  
  syncOptions:
    - CreateNamespace=true
    - PruneLast=true
    - ApplyOutOfSyncOnly=true
  
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
```

## Step 8: Sync Windows (Optional)

Sync windows allow you to restrict when automated syncs can happen.

```yaml
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    
    # Only allow sync during business hours
    syncWindows:
    - kind: allow
      schedule: '0 9-17 * * *'  # 9 AM to 5 PM
      duration: 8h
      applications:
      - '*'
      manualSync: true
```

**Not recommended for beginners**, but useful for:
- Production deployments
- Maintenance windows
- Compliance requirements

## Verification Commands

### Check Sync Policy

```bash
argocd app get <app-name> | grep -A 5 "Sync Policy"
```

### Check Sync Status

```bash
argocd app get <app-name> | grep "Sync Status"
```

### Watch for Changes

```bash
watch -n 5 "argocd app get <app-name> | grep -E '(Sync|Health)'"
```

### View Recent Syncs

```bash
argocd app history <app-name>
```

### Check Events

```bash
kubectl get events -n default --sort-by='.lastTimestamp' | grep <app-name>
```

## Troubleshooting

### Issue: Auto-Sync Not Working

**Check sync policy**:
```bash
argocd app get <app-name> | grep "Sync Policy"
```

Should show `Automated`.

**Check for errors**:
```bash
argocd app get <app-name>
# Look for error messages
```

**Force refresh**:
```bash
argocd app get <app-name> --refresh
```

### Issue: Self-Heal Not Reverting Changes

**Check self-heal is enabled**:
```bash
argocd app get <app-name> | grep -A 3 "Sync Policy"
```

Should show `Self Heal: true`.

**Check sync interval** (default is 3 minutes):
```bash
# It may take a few minutes for Argo CD to detect drift
```

**Check application controller logs**:
```bash
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller --tail=50
```

### Issue: Prune Deleting Wrong Resources

**Check resource labels**:
```bash
kubectl get <resource> -n <namespace> --show-labels
```

**Verify tracking labels**:
```yaml
# Argo CD tracks resources with this label:
app.kubernetes.io/instance: <app-name>
```

**Exclude resources from pruning**:
```yaml
# Add annotation to resource:
metadata:
  annotations:
    argocd.argoproj.io/sync-options: Prune=false
```

## Best Practices

### Development Environment

```yaml
# Fast feedback, aggressive automation
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

✅ Good for: Dev, Test environments  
❌ Bad for: Production

### Staging Environment

```yaml
# Automated but more controlled
syncPolicy:
  automated:
    prune: true
    selfHeal: false  # Allow manual changes for debugging
```

✅ Good for: Staging, Pre-prod  
⚠️ Review: Before promoting to prod

### Production Environment

```yaml
# Manual control, maximum safety
syncPolicy:
  automated:
    prune: false
    selfHeal: false
  # All syncs must be manual
```

✅ Good for: Production  
🎯 Benefit: Full control and review

**Or with automated but careful settings**:
```yaml
syncPolicy:
  automated:
    prune: false     # Don't auto-delete in prod!
    selfHeal: true   # Prevent drift
```

## When to Use What

| Feature | Dev | Stage | Prod | Use Case |
|---------|-----|-------|------|----------|
| Auto-Sync | ✅ | ✅ | ⚠️ | Fast deployment |
| Self-Heal | ✅ | ⚠️ | ✅ | Prevent drift |
| Prune | ✅ | ⚠️ | ❌ | Clean deletion |
| Manual Sync | ❌ | ⚠️ | ✅ | Controlled changes |

Legend:
- ✅ Recommended
- ⚠️ Use with caution
- ❌ Not recommended

## Cleanup

```bash
# Delete test applications
argocd app delete nginx-auto --yes
argocd app delete test-selfheal --yes
argocd app delete test-prune --yes
argocd app delete my-app --yes
```

## What You've Accomplished

✅ Enabled automated synchronization  
✅ Configured prune for automatic cleanup  
✅ Enabled self-heal for drift prevention  
✅ Tested self-heal in action  
✅ Created applications via YAML manifests  
✅ Learned sync options and best practices  

## Key Concepts

1. **Auto-Sync**: Argo CD automatically applies Git changes
2. **Prune**: Automatically deletes resources removed from Git
3. **Self-Heal**: Automatically reverts manual cluster changes
4. **Sync Options**: Fine-tune sync behavior
5. **Different strategies for different environments**

## Next Steps

Continue to:
- **[Lab 4: Using Kustomize](./lab-04.md)** - Multi-environment configuration management
- **[Section 6: Kustomize with Argo CD](../sections/section-06/README.md)** - Learn advanced patterns

## Quick Reference

```bash
# Create with auto-sync
argocd app create <name> --sync-policy automated

# Enable self-heal
argocd app set <name> --self-heal

# Enable prune
argocd app set <name> --auto-prune

# Create from YAML
kubectl apply -f application.yaml

# Check sync policy
argocd app get <name> | grep "Sync Policy"
```

---

**Excellent work!** You now understand true GitOps automation with Argo CD.
