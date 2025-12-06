# Lecture 5.1: Manual vs Auto Sync
Prune & Self-Heal
Sync Options & Best Practices

## Introduction

In GitOps, controlling when and how deployments happen is critical. This lecture explains the difference between manual and automated sync, helping you choose the right policy for each environment.

## Manual Sync

**What it is**: Deployments require explicit approval

**Configuration**:
\`\`\`yaml
syncPolicy: {}  # No automated section = manual sync
\`\`\`

**How it works**:
1. Git changes detected → Application shows OutOfSync
2. Human reviews changes
3. Human triggers sync (UI or CLI)
4. Changes applied to cluster

**Pros**:
- ✅ Full control over deployments
- ✅ Changes reviewed before applying
- ✅ Suitable for production
- ✅ Prevents accidental deployments

**Cons**:
- ❌ Requires manual intervention
- ❌ Slower feedback loop
- ❌ Requires monitoring for OutOfSync

**Best for**: Production environments, critical applications

## Automated Sync

**What it is**: Argo CD automatically deploys Git changes

**Configuration**:
\`\`\`yaml
syncPolicy:
  automated: {}
\`\`\`

**How it works**:
1. Git changes detected → Application shows OutOfSync
2. Argo CD waits ~5 seconds (debounce)
3. Automatically triggers sync
4. Changes applied to cluster

**Pros**:
- ✅ Fast feedback (changes deploy within minutes)
- ✅ No manual intervention needed
- ✅ True continuous deployment
- ✅ Great for dev/staging

**Cons**:
- ❌ Less control
- ❌ Bad changes deploy automatically
- ❌ May deploy too frequently

**Best for**: Development, staging, test environments

## Comparison

| Aspect | Manual Sync | Automated Sync |
|--------|-------------|----------------|
| Control | High | Low |
| Speed | Slow | Fast |
| Safety | High | Medium |
| Effort | High | Low |
| Use Case | Production | Dev/Staging |

## Setting Sync Policy

### Via CLI

\`\`\`bash
# Enable auto-sync
argocd app set <app-name> --sync-policy automated

# Disable auto-sync (make it manual)
argocd app set <app-name> --sync-policy none
\`\`\`

### Via UI

1. Open application
2. Click "App Details" button
3. Click "Edit" next to Sync Policy
4. Toggle "Automatic"
5. Save

### Via YAML

\`\`\`yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  syncPolicy:
    automated: {}  # Enable auto-sync
    # OR omit this section for manual sync
\`\`\`

## Testing Sync Policy

### Test Manual Sync

1. Create app with manual sync
2. Make Git change
3. Observe: App shows OutOfSync
4. Must manually sync

### Test Auto Sync

1. Create app with auto-sync
2. Make Git change
3. Wait ~3 minutes
4. Observe: Auto deploys!

## Best Practices

✅ **Use auto-sync for dev** - fast iteration
✅ **Use manual sync for prod** - safety first
✅ **Review changes always** - even with auto-sync
✅ **Monitor sync status** - set up alerts
✅ **Document policy** - team should know

## Key Takeaways

✅ **Manual sync** = control and safety
✅ **Auto sync** = speed and automation
✅ **Choose based on environment** risk
✅ **Can change policy** anytime
✅ **Both valid** for different scenarios

## Next Steps

Continue to [Lecture 5.2: Prune & Self-Heal](./lecture-5.2.md) to learn about advanced auto-sync options!)

