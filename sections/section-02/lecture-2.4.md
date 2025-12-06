# Lecture 2.4: Sync, Health, and Drift

## Introduction

In this lecture, we'll explore three critical concepts that define the operational state of your applications in Argo CD: Sync (is the app deployed correctly?), Health (is the app running correctly?), and Drift (have things changed unexpectedly?).

## Understanding Sync

### What is Sync?

**Sync** is the process of applying the desired state from Git to the live cluster. It answers the question: "Does what's running match what's in Git?"

### Sync Status

Argo CD applications can be in one of three sync states:

#### 1. Synced ✅

**Meaning**: Live cluster state matches Git desired state

**Example**:
```
Git says: 3 replicas, image v1.0.0
Cluster has: 3 replicas, image v1.0.0
Status: Synced ✅
```

**What this means**:
- All resources match Git
- No pending changes
- Application is at desired state

**Note**: Synced ≠ Healthy! The app can be synced but unhealthy.

#### 2. OutOfSync ⚠️

**Meaning**: Live cluster state differs from Git desired state

**Example**:
```
Git says: 3 replicas, image v2.0.0
Cluster has: 3 replicas, image v1.0.0
Status: OutOfSync ⚠️
```

**Common causes**:
- Git changed, sync not performed yet
- Manual kubectl changes to cluster
- Resource deleted from cluster
- Configuration drift

**What to do**:
- Review the diff
- Sync if changes are desired
- Investigate if unexpected

#### 3. Unknown ❓

**Meaning**: Cannot determine sync status

**Common causes**:
- Permission issues accessing Git
- Permission issues accessing cluster
- Resources still being created
- Temporary API errors

**What to do**:
- Check Application events
- Verify repository credentials
- Verify cluster connectivity
- Check Argo CD logs

### Sync Operations

#### Manual Sync

Requires explicit action to sync:

```bash
# CLI
argocd app sync myapp

# UI
Click "Sync" button

# kubectl
kubectl patch application myapp -n argocd \
  --type merge \
  -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{}}}'
```

**Use case**: Production environments where changes need approval

#### Automated Sync

Syncs automatically when OutOfSync is detected:

```yaml
syncPolicy:
  automated:
    prune: false
    selfHeal: false
```

**How it works**:
1. Argo CD detects OutOfSync
2. Waits a short period (debounce)
3. Automatically triggers sync
4. No human intervention needed

**Use case**: Dev/staging environments for fast feedback

### Sync Options

#### Prune

Delete resources that exist in cluster but not in Git:

```yaml
syncPolicy:
  automated:
    prune: true
```

**Example**:
```
Git: deployment.yaml, service.yaml
Cluster: deployment.yaml, service.yaml, configmap.yaml
Action: Delete configmap.yaml (it's not in Git)
```

**⚠️ Warning**: Be careful with prune in production!

#### Self-Heal

Revert manual cluster changes to match Git:

```yaml
syncPolicy:
  automated:
    selfHeal: true
```

**Example**:
```
1. Git says: 3 replicas
2. Someone runs: kubectl scale deployment myapp --replicas=5
3. Argo CD detects: Cluster has 5, Git has 3
4. Self-heal triggers: Scale back to 3
```

**Use case**: Prevent configuration drift

### Sync Phases

A sync operation goes through several phases:

```
PreSync → Sync → PostSync
```

#### 1. PreSync

Resources with sync phase PreSync run first:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
    argocd.argoproj.io/hook: PreSync
```

**Example**: Database schema migration

#### 2. Sync

Main application resources are applied:

```yaml
# No special annotation needed
# This is the default
```

#### 3. PostSync

Resources run after sync completes:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/hook: PostSync
```

**Example**: Smoke tests, notifications

## Understanding Health

### What is Health?

**Health** indicates whether deployed resources are functioning correctly. It answers: "Is the app running properly?"

### Health Status

#### 1. Healthy 💚

**Meaning**: All resources are running correctly

**Examples**:
- Deployment: All replicas available
- Service: Endpoints exist
- StatefulSet: All replicas ready
- Job: Successfully completed

```bash
$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
myapp-7d8b9c-abc        1/1     Running   0          5m
myapp-7d8b9c-def        1/1     Running   0          5m
myapp-7d8b9c-ghi        1/1     Running   0          5m

Status: Healthy 💚
```

#### 2. Progressing 🟡

**Meaning**: Resources are being created or updated

**Examples**:
- Deployment: Rolling update in progress
- Pods: Container creating
- PVC: Provisioning storage

```bash
$ kubectl get pods
NAME                    READY   STATUS              RESTARTS   AGE
myapp-7d8b9c-abc        1/1     Running             0          5m
myapp-new-xyz           0/1     ContainerCreating   0          10s

Status: Progressing 🟡
```

**Note**: This is normal during deployments

#### 3. Degraded 🔴

**Meaning**: Resources have problems

**Examples**:
- Pods: CrashLoopBackOff
- Deployment: Not enough replicas available
- Service: No endpoints
- Job: Failed

```bash
$ kubectl get pods
NAME                    READY   STATUS             RESTARTS   AGE
myapp-7d8b9c-abc        0/1     CrashLoopBackOff   5          5m
myapp-7d8b9c-def        0/1     ImagePullBackOff   0          5m

Status: Degraded 🔴
```

**Action**: Investigate logs and events immediately

#### 4. Suspended ⏸️

**Meaning**: Resources intentionally suspended

**Examples**:
- CronJob: Suspended
- Deployment: Replicas set to 0

```yaml
spec:
  suspend: true
```

**Note**: This is intentional, not an error

#### 5. Missing ❌

**Meaning**: Resources should exist but don't

**Example**:
```
Git defines: Deployment, Service
Cluster has: (nothing)
Status: Missing ❌
```

**Cause**: Resources not yet created or were deleted

#### 6. Unknown ❓

**Meaning**: Cannot determine health

**Causes**:
- Resource type doesn't have health check
- Permission issues
- Custom resources without health check

### Health Assessment

Argo CD assesses health differently for each resource type:

#### Deployments

```go
if availableReplicas >= desiredReplicas:
    return Healthy
elif progressingCondition:
    return Progressing
else:
    return Degraded
```

#### Pods

```go
if phase == "Running" and ready:
    return Healthy
elif phase == "Pending":
    return Progressing
elif phase in ["Failed", "CrashLoopBackOff", "ImagePullBackOff"]:
    return Degraded
```

#### Services

```go
if type == "LoadBalancer":
    if loadBalancer.ingress exists:
        return Healthy
    else:
        return Progressing
elif endpoints exist:
    return Healthy
else:
    return Degraded
```

#### Jobs

```go
if succeeded >= completions:
    return Healthy
elif failed > backoffLimit:
    return Degraded
else:
    return Progressing
```

### Custom Health Checks

For custom resources, define custom health checks:

```lua
-- In argocd-cm ConfigMap
resource.customizations.health.cert-manager.io_Certificate: |
  hs = {}
  if obj.status ~= nil then
    if obj.status.conditions ~= nil then
      for i, condition in ipairs(obj.status.conditions) do
        if condition.type == "Ready" and condition.status == "True" then
          hs.status = "Healthy"
          hs.message = "Certificate is ready"
          return hs
        end
      end
    end
  end
  hs.status = "Progressing"
  hs.message = "Waiting for certificate"
  return hs
```

## Understanding Drift

### What is Drift?

**Drift** occurs when the live cluster state diverges from Git without an intentional change in Git.

### Common Causes of Drift

#### 1. Manual kubectl Changes

```bash
# Someone runs this
kubectl scale deployment myapp --replicas=10

# Now cluster has 10 replicas but Git says 3
# This is drift! 📊
```

#### 2. Horizontal Pod Autoscaler (HPA)

```yaml
# Git defines
spec:
  replicas: 3

# HPA changes to
spec:
  replicas: 8  # Based on load

# This is expected drift
```

#### 3. Operators Modifying Resources

```yaml
# You define
apiVersion: v1
kind: Service
spec:
  type: LoadBalancer

# Cloud controller adds
spec:
  loadBalancerIP: 10.0.0.1
  loadBalancerSourceRanges:
    - 0.0.0.0/0

# This is acceptable drift
```

#### 4. Cluster Autoscaler

Node resources change dynamically—this is expected.

#### 5. Emergency Hotfixes

```bash
# Production is down! Quick fix:
kubectl set image deployment/myapp app=myapp:v1.2.1-hotfix

# This creates drift but was necessary
# Remember to update Git afterward!
```

### Detecting Drift

#### Via UI

1. Application shows "OutOfSync"
2. Click "App Diff"
3. Review differences
4. Identify unexpected changes

#### Via CLI

```bash
# Check sync status
argocd app get myapp | grep "Sync Status"

# View detailed diff
argocd app diff myapp

# Example output showing drift:
===== apps/Deployment myapp ======
--- Git (desired)
+++ Cluster (live)
@@ -10,7 +10,7 @@
-  replicas: 3
+  replicas: 10
```

#### Via Prometheus Metrics

```prometheus
# Monitor OutOfSync applications
argocd_app_info{sync_status="OutOfSync"}

# Alert on prolonged OutOfSync
alert: ApplicationOutOfSync
expr: argocd_app_info{sync_status="OutOfSync"} > 0
for: 15m
```

### Handling Drift

#### Option 1: Prevent Drift (Self-Heal)

```yaml
syncPolicy:
  automated:
    selfHeal: true
```

**Behavior**: Argo CD automatically reverts manual changes

**Best for**: Dev/staging environments

**⚠️ Warning**: Can interfere with debugging

#### Option 2: Accept Drift (Ignore Differences)

```yaml
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas
```

**Behavior**: Argo CD ignores specific fields

**Best for**: HPA-managed replicas, operator-managed fields

#### Option 3: Sync to Fix Drift

```bash
# Review diff
argocd app diff myapp

# If acceptable, sync
argocd app sync myapp
```

**Behavior**: Overwrite cluster with Git

**Best for**: One-time drift correction

#### Option 4: Update Git to Match Cluster

```bash
# Export current cluster state
kubectl get deployment myapp -o yaml > deployment.yaml

# Edit and commit to Git
git add deployment.yaml
git commit -m "Accept manual scaling change"
git push
```

**Behavior**: Make Git match reality

**Best for**: When cluster change is desired

### Drift Prevention Strategies

#### 1. RBAC Restrictions

Limit who can make direct cluster changes:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developers
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: read-only
subjects:
- kind: Group
  name: developers
```

#### 2. Admission Controllers

Use OPA/Kyverno to prevent manual changes:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-gitops-annotation
spec:
  validationFailureAction: enforce
  rules:
  - name: check-for-gitops-annotation
    match:
      resources:
        kinds:
        - Deployment
    validate:
      message: "Deployments must have gitops annotation"
      pattern:
        metadata:
          annotations:
            argocd.argoproj.io/instance: "?*"
```

#### 3. Monitoring and Alerts

Alert on drift:

```yaml
- alert: UnexpectedDrift
  expr: |
    argocd_app_info{sync_status="OutOfSync", project!="dev"} > 0
  for: 10m
  annotations:
    summary: "Application {{ $labels.name }} has unexpected drift"
```

#### 4. Regular Audits

```bash
# Weekly drift check
for app in $(argocd app list -o name); do
  status=$(argocd app get $app | grep "Sync Status")
  echo "$app: $status"
done
```

## Sync, Health, and Drift Together

### Application State Matrix

| Sync Status | Health Status | Drift | Meaning | Action |
|-------------|---------------|-------|---------|--------|
| Synced | Healthy | No | ✅ Perfect! | Monitor |
| Synced | Degraded | No | ⚠️ App deployed correctly but not running | Fix app issues |
| OutOfSync | Healthy | Yes | ⚠️ Drift detected, but app running | Investigate & sync |
| OutOfSync | Degraded | Yes | 🔴 Drift + broken app | Fix immediately |
| OutOfSync | Progressing | No | 🟡 Sync in progress | Wait |

### Real-World Scenario

**Initial State**:
```
Sync: Synced ✅
Health: Healthy 💚
Action: All good! ☺️
```

**Git Updated**:
```
Sync: OutOfSync ⚠️ (Git has v2.0.0, cluster has v1.0.0)
Health: Healthy 💚 (v1.0.0 still running fine)
Action: Review diff, then sync
```

**During Sync**:
```
Sync: Synced ✅ (sync completed)
Health: Progressing 🟡 (rolling update in progress)
Action: Wait for rollout to complete
```

**After Sync**:
```
Sync: Synced ✅
Health: Healthy 💚
Action: Success! ☺️
```

**Manual Change** (someone scales deployment):
```
Sync: OutOfSync ⚠️ (drift detected)
Health: Healthy 💚 (app still running)
Action: Decide: revert or accept?
```

## Best Practices

### Sync

✅ Use auto-sync in dev/staging  
✅ Use manual sync in production  
✅ Always review diff before syncing  
✅ Use sync waves for complex deployments  
⚠️ Be careful with prune

### Health

✅ Monitor health status continuously  
✅ Set up alerts for degraded applications  
✅ Define custom health checks for CRDs  
✅ Understand health is independent from sync

### Drift

✅ Enable self-heal in dev environments  
✅ Use ignoreDifferences for expected drift (HPA)  
✅ Investigate unexpected drift immediately  
✅ Prevent drift with RBAC and admission controllers  
✅ Git is always the source of truth

## Key Takeaways

✅ **Sync**: Is the app deployed correctly?  
✅ **Health**: Is the app running correctly?  
✅ **Drift**: Are there unexpected changes?  
✅ **Synced ≠ Healthy**: These are independent  
✅ **Monitor all three**: Complete picture requires all  
✅ **Git wins**: In GitOps, Git is the authority

## Common Questions

**Q: Can an app be Synced but Unhealthy?**  
A: Yes! Sync means it matches Git. Health means it's running correctly.

**Q: Should I always enable self-heal?**  
A: Good for dev/staging. Use carefully in production as it can interfere with debugging.

**Q: How quickly does Argo CD detect drift?**  
A: Default is 3 minutes. Use webhooks for instant detection.

**Q: What if I need to make emergency changes to production?**  
A: Make the change, then immediately update Git to match. Document why.

**Q: Can I sync only specific resources?**  
A: Yes, use selective sync in UI or `argocd app sync myapp --resource` in CLI.

## Next Steps

Congratulations! You've completed Section 2. You now understand Argo CD architecture and core concepts.

Continue to [Section 3: Lab Setup](../section-03/README.md) to set up your environment and start hands-on learning.

Don't forget to complete the [Section 2 Quiz](../../quizzes/section-02-quiz.md)!
