# Section 5: Sync Policies Deep Dive

## Overview

Master Argo CD's synchronization policies to control how and when your applications are deployed. This section covers the critical differences between manual and automated sync, pruning, self-healing, and best practices for production environments.

## Learning Objectives

By the end of this section, you will:
- Understand the difference between manual and automated sync
- Know when to use prune and self-heal options
- Configure sync options for different scenarios
- Handle drift safely in production environments
- Implement appropriate sync policies per environment

## Lectures

1. [5.1 Manual vs Auto Sync](./lecture-5.1.md)
2. [5.2 Prune & Self-Heal](./lecture-5.2.md)
3. [5.3 Sync Options & Best Practices](./lecture-5.3.md)
4. [5.4 Handling Drift Safely](./lecture-5.4.md)

## Hands-On Lab

Complete **[Lab 3: Automated Sync and Self-Heal](../../labs/lab-03.md)** which covers all topics in this section.

## Key Takeaways

- **Manual Sync**: Full control, best for production
- **Automated Sync**: Fast feedback, best for dev/staging
- **Prune**: Automatically deletes resources removed from Git
- **Self-Heal**: Automatically reverts manual cluster changes
- **Different environments need different policies**

## Sync Policy Matrix

| Environment | Auto-Sync | Self-Heal | Prune | Reasoning |
|-------------|-----------|-----------|-------|-----------|
| Development | ✅ Yes | ✅ Yes | ✅ Yes | Fast iteration, prevent manual changes |
| Staging | ✅ Yes | ⚠️ Maybe | ⚠️ Maybe | Semi-automated, allow debugging |
| Production | ❌ No | ✅ Yes | ❌ No | Manual approval, prevent drift, safe deletes |

## Sync Policy Examples

### Development Environment

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true
```

**Characteristics**:
- Changes sync automatically
- Manual changes reverted
- Removed resources deleted
- Fast feedback loop

### Staging Environment

```yaml
syncPolicy:
  automated:
    prune: false
    selfHeal: false
  syncOptions:
    - CreateNamespace=true
```

**Characteristics**:
- Changes sync automatically
- Can make manual changes for debugging
- Resources not auto-deleted
- Balance of automation and control

### Production Environment

```yaml
syncPolicy:
  automated: null  # Manual sync only
  syncOptions:
    - CreateNamespace=false
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
```

**Characteristics**:
- Manual sync required
- Full review before deployment
- Retry logic for transient failures
- Maximum safety

## Understanding Sync Status

### OutOfSync

**Meaning**: Desired state (Git) differs from live state (cluster)

**Causes**:
- Git changes not yet synced
- Manual cluster changes
- Sync policy prevents auto-sync

**Action**: Review diff, then sync

### Synced

**Meaning**: Live state matches desired state

**Note**: Doesn't mean healthy! Check health status too.

### Unknown

**Meaning**: Cannot determine sync status

**Causes**:
- Permission issues
- API errors
- Resources being created

## Understanding Health Status

### Healthy

Resources are running correctly

### Progressing

Resources are being created/updated

### Degraded

Resources have issues (pods crashing, etc.)

### Suspended

Resources intentionally suspended

### Missing

Resources don't exist in cluster

## Sync Options Deep Dive

### CreateNamespace

```yaml
syncOptions:
  - CreateNamespace=true
```

Automatically creates namespace if it doesn't exist.

### PrunePropagationPolicy

```yaml
syncOptions:
  - PrunePropagationPolicy=foreground
```

Controls how resources are deleted:
- `foreground`: Delete dependents first
- `background`: Delete resource, dependents async
- `orphan`: Delete resource, keep dependents

### PruneLast

```yaml
syncOptions:
  - PruneLast=true
```

Prune resources only after other resources are healthy.

### Validate

```yaml
syncOptions:
  - Validate=false
```

Skip kubectl validation (use carefully).

### ApplyOutOfSyncOnly

```yaml
syncOptions:
  - ApplyOutOfSyncOnly=true
```

Only sync resources that are out-of-sync (performance optimization).

## Drift Detection and Remediation

### What is Drift?

Drift occurs when live cluster state differs from Git desired state.

**Common causes**:
- Manual kubectl changes
- Operators modifying resources
- Horizontal Pod Autoscaler (HPA) changing replicas
- Cluster autoscaling

### Detecting Drift

```bash
# Check sync status
argocd app get <app-name> | grep "Sync Status"

# View diff
argocd app diff <app-name>

# Refresh to latest
argocd app get <app-name> --refresh
```

### Handling Drift

**Option 1: Revert (Self-Heal)**
```yaml
syncPolicy:
  automated:
    selfHeal: true
```

**Option 2: Accept and Update Git**
```bash
# If change is desired, update Git
kubectl get deployment myapp -o yaml > deployment.yaml
# Edit and commit to Git
```

**Option 3: Ignore Differences**
```yaml
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas  # Ignore HPA changes
```

## Best Practices

### 1. Match Policy to Environment Risk

Lower risk = more automation  
Higher risk = more control

### 2. Use Self-Heal in Production

Prevents configuration drift while allowing manual review of intended changes.

### 3. Be Careful with Prune

In production, manually review deletions before enabling auto-prune.

### 4. Configure Retry Logic

```yaml
retry:
  limit: 5
  backoff:
    duration: 5s
    factor: 2
    maxDuration: 3m
```

Handles transient failures gracefully.

### 5. Use Sync Windows for Production

```yaml
syncWindows:
- kind: allow
  schedule: '0 9-17 * * 1-5'  # Business hours, weekdays
  duration: 8h
```

Control when syncs can occur.

## Common Pitfalls

❌ **Enabling prune without testing**  
✅ Test prune in dev first

❌ **Auto-sync to production**  
✅ Require manual approval for prod

❌ **Ignoring drift warnings**  
✅ Investigate and fix root cause

❌ **Same policy for all environments**  
✅ Tailor policies to environment needs

## Next Steps

After completing this section and Lab 3:
- You'll master sync policies
- You'll know how to handle drift
- You'll be ready to learn Kustomize for multi-environment configs
- Continue to [Section 6: Kustomize with Argo CD](../section-06/README.md)

## Quiz

Complete the [Section 5 Quiz](../../quizzes/section-05-quiz.md) to test your understanding.

## Additional Resources

- [Sync Policies Documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/)
- [Sync Options Reference](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/)
