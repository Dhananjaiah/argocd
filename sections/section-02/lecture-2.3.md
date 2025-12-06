# Lecture 2.3: Desired vs Live State

## Introduction

The core concept of GitOps and Argo CD is maintaining the desired state in Git and continuously reconciling the live state in Kubernetes. Understanding the difference between these two states and how Argo CD manages them is fundamental to mastering GitOps.

## What is State?

### Desired State

The **desired state** is what you **want** your application to look like. In GitOps, this is defined in Git.

**Characteristics**:
- ✅ Declarative (YAML, not commands)
- ✅ Version controlled in Git
- ✅ Single source of truth
- ✅ Auditable and traceable
- ✅ Can be reviewed before deployment

**Example**:
```yaml
# In Git: deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3        # DESIRED: 3 replicas
  template:
    spec:
      containers:
      - name: app
        image: myapp:v1.0.0    # DESIRED: v1.0.0
```

### Live State

The **live state** is what **actually exists** in your Kubernetes cluster right now.

**Characteristics**:
- ✅ Real-time cluster resources
- ✅ Retrieved from Kubernetes API
- ✅ May differ from desired state
- ✅ Includes runtime changes
- ✅ Reflects actual running workloads

**Example**:
```bash
# In Cluster: actual running deployment
kubectl get deployment myapp -o yaml

# Shows:
# replicas: 2              # LIVE: only 2 replicas running
# image: myapp:v0.9.0      # LIVE: old version still running
```

## Comparing Desired vs Live State

### When States Match

```
Git (Desired State)          Kubernetes (Live State)
┌─────────────────┐         ┌─────────────────┐
│ replicas: 3     │         │ replicas: 3     │
│ image: v1.0.0   │    =    │ image: v1.0.0   │
│ cpu: 100m       │         │ cpu: 100m       │
└─────────────────┘         └─────────────────┘
        
Status: ✅ Synced
```

### When States Don't Match

```
Git (Desired State)          Kubernetes (Live State)
┌─────────────────┐         ┌─────────────────┐
│ replicas: 3     │         │ replicas: 2     │
│ image: v1.0.0   │    ≠    │ image: v0.9.0   │
│ cpu: 100m       │         │ cpu: 50m        │
└─────────────────┘         └─────────────────┘
        
Status: ⚠️ OutOfSync
```

## How Argo CD Tracks State

### State Discovery Process

```
1. Read Desired State
   ├─> Connect to Git repository
   ├─> Checkout targetRevision (branch/tag/commit)
   ├─> Read manifests from path
   └─> Generate manifests (Helm/Kustomize if needed)

2. Read Live State
   ├─> Connect to Kubernetes cluster
   ├─> Query resources matching Application
   ├─> Get current resource definitions
   └─> Read resource status

3. Compare States
   ├─> Normalize both states
   ├─> Calculate diff
   └─> Determine sync status
```

### Continuous Monitoring

Argo CD doesn't just check once—it continuously monitors:

```
Every 3 minutes (configurable):
├─> Refresh desired state from Git
├─> Query live state from Kubernetes
├─> Compare and update sync status
└─> Update health status

On webhook event:
├─> Immediate refresh
└─> Faster detection of changes
```

## State Comparison Example

### Scenario: Updating Image Version

**Initial State (Synced)**:

Git:
```yaml
image: myapp:v1.0.0
```

Cluster:
```yaml
image: myapp:v1.0.0
```

Status: `Synced` ✅

**After Git Update (OutOfSync)**:

Git:
```yaml
image: myapp:v2.0.0    # Updated in Git
```

Cluster:
```yaml
image: myapp:v1.0.0    # Still old version
```

Status: `OutOfSync` ⚠️

**After Sync (Synced Again)**:

Git:
```yaml
image: myapp:v2.0.0
```

Cluster:
```yaml
image: myapp:v2.0.0    # Now updated
```

Status: `Synced` ✅

## Viewing State in Argo CD

### Via UI

1. Open Application in UI
2. Click "App Diff" button
3. See side-by-side comparison:

```
Desired (Git)              Live (Cluster)
───────────────           ────────────────
replicas: 3               replicas: 2
                          │
                          └─> Shows difference
```

### Via CLI

```bash
# View current state
argocd app get myapp

# View detailed diff
argocd app diff myapp

# Example output:
===== apps/Deployment myapp ======
--- /tmp/desired-state.yaml
+++ /tmp/live-state.yaml
@@ -10,7 +10,7 @@
-  replicas: 3
+  replicas: 2
```

### Via kubectl

```bash
# Get Application status
kubectl get application myapp -n argocd -o yaml

# Check sync status
kubectl get application myapp -n argocd -o jsonpath='{.status.sync.status}'
# Output: Synced or OutOfSync
```

## State Drift Scenarios

### Scenario 1: Git Changed, Cluster Unchanged

**What happened**: Developer pushed changes to Git

```
Git: Updated → Cluster: Old → Status: OutOfSync
```

**Action**: Sync to apply changes

```bash
argocd app sync myapp
```

### Scenario 2: Cluster Changed Manually

**What happened**: Someone ran `kubectl` directly

```
Git: Original → Cluster: Modified → Status: OutOfSync
```

**Action**: Depends on sync policy
- Manual sync: Wait for review
- Auto sync + self-heal: Auto-revert to Git

### Scenario 3: Both Changed Differently

**What happened**: Git updated, cluster also changed manually

```
Git: Version A → Cluster: Version B → Status: OutOfSync
```

**Action**: Sync will overwrite cluster with Git (Git wins)

### Scenario 4: External Controllers

**What happened**: HPA changed replica count

```
Git: replicas: 3 → Cluster: replicas: 5 (HPA scaled) → Status: ?
```

**Action**: Configure ignoreDifferences:

```yaml
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas
```

## State Normalization

Argo CD normalizes both states before comparing to avoid false positives.

### What Gets Normalized?

1. **Field ordering**: Order doesn't matter
2. **Default values**: Kubernetes defaults are added
3. **Read-only fields**: Status fields are ignored
4. **Annotations**: Some auto-generated annotations ignored
5. **Last-applied-configuration**: kubectl annotations ignored

### Example

These are considered **identical**:

**Desired State**:
```yaml
containers:
- name: app
  image: nginx:1.25
  ports:
  - containerPort: 80
```

**Live State** (with Kubernetes defaults):
```yaml
containers:
- name: app
  image: nginx:1.25
  imagePullPolicy: IfNotPresent    # K8s default
  ports:
  - containerPort: 80
    protocol: TCP                   # K8s default
  resources: {}                     # K8s default
  terminationMessagePath: /dev/termination-log  # K8s default
  terminationMessagePolicy: File
```

## Resource Tracking

Argo CD tracks which resources belong to an Application using labels:

```yaml
metadata:
  labels:
    app.kubernetes.io/instance: myapp
```

### Tracking Methods

1. **Label-based** (default):
```yaml
labels:
  app.kubernetes.io/instance: myapp
```

2. **Annotation-based**:
```yaml
annotations:
  argocd.argoproj.io/tracking-id: myapp:/Deployment:default/myapp
```

3. **Annotation + Label**:
Best of both worlds

### Why Tracking Matters

Without proper tracking:
- Argo CD can't detect live state
- Orphaned resources remain in cluster
- Prune operations won't work
- Health assessment fails

## Advanced State Concepts

### Ignored Differences

Tell Argo CD to ignore specific fields:

```yaml
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas            # Ignore replica count
  - group: apps
    kind: StatefulSet
    jqPathExpressions:
    - .spec.volumeClaimTemplates[]?.metadata.annotations
```

**Use cases**:
- HPA-managed replica counts
- Operator-managed fields
- Runtime-only annotations

### Respect Ignore Differences

```yaml
syncOptions:
  - RespectIgnoreDifferences=true
```

Ensures ignored fields aren't touched during sync.

### Comparing Specific Revisions

```bash
# Compare specific Git revision vs cluster
argocd app diff myapp --revision <commit-sha>

# Compare two Git revisions
git diff v1.0.0..v2.0.0 kubernetes/
```

## Best Practices

### 1. Git is the Source of Truth

✅ Always update Git first, then sync  
❌ Don't make manual changes expecting them to persist

### 2. Use Auto-Sync with Self-Heal

✅ Prevents drift in lower environments  
⚠️ Use carefully in production

### 3. Configure Ignore Differences Wisely

✅ Ignore HPA-managed fields  
✅ Ignore operator-managed fields  
❌ Don't ignore important configuration

### 4. Monitor Sync Status

✅ Set up alerts for prolonged OutOfSync  
✅ Investigate unexpected OutOfSync status  
✅ Regular sync status reviews

### 5. Understand the Diff

✅ Always review diff before syncing  
✅ Understand what will change  
❌ Don't blindly sync without reviewing

## Key Takeaways

✅ **Desired State**: What's defined in Git  
✅ **Live State**: What exists in Kubernetes  
✅ **Synced**: States match  
✅ **OutOfSync**: States differ  
✅ **GitOps**: Git is always the source of truth  
✅ **Continuous Reconciliation**: Argo CD constantly compares states

## Common Questions

**Q: How often does Argo CD check for state differences?**  
A: Every 3 minutes by default (configurable with `timeout.reconciliation`).

**Q: Can live state changes survive a sync?**  
A: Only if you use `ignoreDifferences` or disable self-heal.

**Q: What happens if both Git and cluster change?**  
A: Git wins. Sync overwrites cluster with Git state.

**Q: How do I handle HPA without constant OutOfSync?**  
A: Use `ignoreDifferences` to ignore the `replicas` field.

**Q: Can I preview what will change before syncing?**  
A: Yes, use `argocd app diff` or check the UI's diff view.

## Next Steps

Now that you understand desired vs live state, continue to [Lecture 2.4: Sync, Health, and Drift](./lecture-2.4.md) to learn about sync operations and health monitoring.
