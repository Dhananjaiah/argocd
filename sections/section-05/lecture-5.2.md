# Lecture 5.2: Prune & Self-Heal

## Introduction

Automated sync is powerful, but prune and self-heal take it to the next level. These options control what happens to extra resources and manual changes.

## Prune

**What it does**: Automatically deletes resources that exist in cluster but not in Git

**Configuration**:
\`\`\`yaml
syncPolicy:
  automated:
    prune: true
\`\`\`

**How it works**:
1. Argo CD detects resource in cluster
2. Resource doesn't exist in Git
3. Resource is deleted from cluster

**Example**:
\`\`\`
Git: deployment.yaml, service.yaml
Cluster: deployment.yaml, service.yaml, configmap.yaml
Result: configmap.yaml is DELETED
\`\`\`

**Use Cases**:
- Clean up old resources
- Remove deprecated configs
- Keep cluster clean

**⚠️ Warning**: Use carefully! Can accidentally delete important resources.

## Self-Heal

**What it does**: Automatically reverts manual cluster changes to match Git

**Configuration**:
\`\`\`yaml
syncPolicy:
  automated:
    selfHeal: true
\`\`\`

**How it works**:
1. Someone makes manual change (kubectl)
2. Argo CD detects drift
3. Argo CD reverts change to match Git

**Example**:
\`\`\`bash
# Git says 3 replicas
# Someone runs: kubectl scale deployment myapp --replicas=10
# Self-heal reverts to 3 replicas
\`\`\`

**Use Cases**:
- Prevent configuration drift
- Enforce Git as source of truth
- Block manual changes

**⚠️ Warning**: Can interfere with debugging!

## Combining Prune and Self-Heal

\`\`\`yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
\`\`\`

**Effect**:
- Auto-deploys Git changes ✅
- Deletes extra resources ✅
- Reverts manual changes ✅
- Fully automated GitOps ✅

**Best for**: Development environments

## Recommended Policies by Environment

### Development
\`\`\`yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
\`\`\`

### Staging
\`\`\`yaml
syncPolicy:
  automated:
    prune: false
    selfHeal: false
\`\`\`

### Production
\`\`\`yaml
syncPolicy: {}  # Manual sync only
\`\`\`

## Testing Self-Heal

\`\`\`bash
# 1. Enable self-heal
argocd app set myapp --self-heal

# 2. Make manual change
kubectl scale deployment myapp --replicas=10

# 3. Wait ~3 minutes
# 4. Check: Should revert to Git value!
kubectl get deployment myapp
\`\`\`

## Key Takeaways

✅ **Prune** removes extra resources
✅ **Self-Heal** enforces Git state
✅ **Use carefully** in production
✅ **Great for dev** environments
✅ **Prevents drift** effectively

## Next Steps

Continue to [Lecture 5.3: Sync Options & Best Practices](./lecture-5.3.md)!)

