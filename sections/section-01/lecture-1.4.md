# Lecture 1.4: Core GitOps Principles & Workflows

**Duration**: 6 minutes

## Instructor Script

"Let's solidify your understanding of GitOps by exploring the four core principles and the workflows that make it all work.

These principles might seem simple, but they're profound. When you follow them consistently, you get an incredibly robust system. Your infrastructure becomes code that's reviewed, versioned, and tested just like your application code. Your operations become reproducible and auditable. And your team can collaborate using tools they already know – Git, PRs, and code review.

The workflow is elegant: write YAML, commit to Git, create a PR, review, merge, and watch it deploy automatically. If something goes wrong, you don't panic and run manual commands – you simply revert the commit and push. That's the power of GitOps.

Let's explore these principles and workflows in detail so you'll be ready to implement them in the upcoming labs."

## The Four Core Principles of GitOps

### 1. Declarative Configuration

**Principle**: The desired state of your system is expressed declaratively.

**What This Means**:
- You describe *what* you want, not *how* to get there
- Use YAML manifests, not bash scripts
- Kubernetes is declarative by nature

**Example**:

❌ **Imperative (Traditional)**:
```bash
kubectl create deployment myapp --image=myapp:v1.0
kubectl scale deployment myapp --replicas=3
kubectl expose deployment myapp --port=80
kubectl set image deployment/myapp myapp=myapp:v2.0
```

✅ **Declarative (GitOps)**:
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
```

**Why It Matters**:
- Complete state in one file
- Easy to review and understand
- Can be version controlled
- Reproducible across environments

### 2. Versioned and Immutable

**Principle**: The desired state is stored in Git and versioned.

**What This Means**:
- Every change is a Git commit
- Full history of all changes
- Easy rollback to any previous state
- Immutable audit trail

**Example**:

```bash
# View history
git log --oneline k8s/deployment.yaml

a3f62d0 Update to version 2.0
e8c9b1a Increase replicas to 3
7f4e82c Initial deployment

# See exact changes
git show 7f4e82c

# Rollback to previous version
git revert a3f62d0
git push
# Argo CD automatically deploys the old version!
```

**Benefits**:
- **Auditability**: Who changed what, when, and why
- **Collaboration**: Code review via Pull Requests
- **Rollback**: Easy revert to any previous state
- **Disaster Recovery**: Entire infrastructure in Git

### 3. Automatically Pulled

**Principle**: Software agents automatically pull the desired state from Git.

**What This Means**:
- Argo CD runs in your cluster
- Continuously polls Git for changes
- Applies changes automatically
- No manual intervention needed

**Example**:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  source:
    repoURL: https://github.com/myorg/myapp
    path: k8s
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  # Argo CD polls this repo every 3 minutes by default
```

**Flow**:
```
Developer → Git commit → Git push → Argo CD polls Git 
→ Detects change → Applies to cluster → Done!
```

**Why Pull vs Push**:
- **Security**: No cluster credentials outside cluster
- **Resilience**: Works even if external systems are down
- **Simplicity**: No need to configure webhooks
- **Multi-cluster**: Easy to manage many clusters

### 4. Continuously Reconciled

**Principle**: Software agents continuously observe actual state and attempt to reconcile with desired state.

**What This Means**:
- Argo CD constantly compares Git (desired) vs Cluster (actual)
- Detects drift automatically
- Can auto-correct drift (self-heal)
- Ensures cluster always matches Git

**Example**:

```bash
# Current state in Git
image: myapp:v2.0

# Someone manually changes cluster
kubectl set image deployment/myapp myapp=myapp:v1.0

# Without selfHeal:
# Argo CD shows "OutOfSync" status
# Manual sync required

# With selfHeal:
# Argo CD detects drift
# Automatically reverts to v2.0
# Cluster matches Git again
```

**Configuration**:
```yaml
syncPolicy:
  automated:
    selfHeal: true    # Auto-revert manual changes
    prune: true       # Auto-delete resources not in Git
```

**Why It Matters**:
- Prevents configuration drift
- Ensures consistency
- Recovers from manual changes
- Maintains desired state

## GitOps Workflow

### Developer Workflow

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  1. Developer makes change locally                  │
│     $ vim k8s/deployment.yaml                       │
│     (change image: myapp:v1.0 → myapp:v2.0)        │
│                                                     │
│  2. Commit and push to Git                         │
│     $ git commit -am "Update to v2.0"              │
│     $ git push origin feature-branch               │
│                                                     │
│  3. Create Pull Request                            │
│     $ gh pr create --title "Deploy v2.0"           │
│                                                     │
│  4. Code Review                                    │
│     - Team reviews the change                       │
│     - CI runs tests                                 │
│     - Approval required                             │
│                                                     │
│  5. Merge to main                                  │
│     $ gh pr merge                                   │
│                                                     │
│  6. Argo CD detects change                         │
│     - Polls Git repository                          │
│     - Detects difference                            │
│     - Syncs to cluster                              │
│                                                     │
│  7. Application updated                            │
│     - New version deployed                          │
│     - Health checks pass                            │
│     - Team notified via Slack                       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Multi-Environment Workflow

```
┌──────────┐      ┌──────────┐      ┌──────────┐
│   Dev    │      │  Stage   │      │   Prod   │
│ Cluster  │      │ Cluster  │      │ Cluster  │
└────┬─────┘      └────┬─────┘      └────┬─────┘
     │                 │                  │
     │ Auto-sync       │ Manual review    │ Manual review
     │                 │                  │
┌────▼─────┐      ┌────▼─────┐      ┌────▼─────┐
│   Dev    │      │  Stage   │      │   Prod   │
│  Branch  │      │  Branch  │      │  Branch  │
└────┬─────┘      └────┬─────┘      └────┬─────┘
     │                 ▲                  ▲
     │                 │                  │
     └─────PR merge────┘                  │
                       │                  │
                       └──────PR merge────┘

Steps:
1. Merge to dev → Auto-deploys to dev cluster
2. Test in dev → Create PR to stage
3. Merge to stage → Manual sync to stage cluster
4. Test in stage → Create PR to prod
5. Merge to prod → Manual sync to prod cluster
```

### Rollback Workflow

```bash
# Problem detected in production

# Option 1: Git revert
git revert HEAD
git push origin main
# Argo CD automatically syncs old version

# Option 2: Argo CD CLI
argocd app rollback myapp
# Reverts to previous successful sync

# Option 3: Argo CD UI
# Click on app → History → Select previous version → Rollback
```

## Best Practices

### 1. **Branch Strategy**

```
main (prod)
  ├── staging
  │     └── develop
  │           └── feature-branches
```

Or per-environment folders:
```
repo/
  ├── environments/
  │     ├── dev/
  │     ├── staging/
  │     └── production/
```

### 2. **PR-Based Promotion**

```bash
# Promote from dev to stage
git checkout staging
git merge dev
git push origin staging
# Create PR for review

# After approval, merge
# Stage environment auto-updates
```

### 3. **Separate App Config from App Code**

```
app-code-repo/           gitops-config-repo/
  ├── src/                 ├── dev/
  ├── tests/               ├── staging/
  └── Dockerfile           └── production/
```

**Why**: Different teams, different access, different update cadence

### 4. **Use Sync Policies Appropriately**

```yaml
# Development: Fast feedback
syncPolicy:
  automated:
    prune: true
    selfHeal: true

# Production: Controlled changes
syncPolicy:
  automated:
    prune: false
    selfHeal: false
  # Manual sync required
```

### 5. **Test Changes in Lower Environments First**

```
Feature → Dev (auto) → Staging (manual) → Prod (manual)
          Test 1       Test 2              Test 3
```

## Common Anti-Patterns

❌ **Making Manual kubectl Changes**
```bash
kubectl edit deployment myapp  # Creates drift!
```
✅ **Update Git Instead**
```bash
vim k8s/deployment.yaml
git commit -am "Update deployment"
git push
```

❌ **Storing Secrets in Git**
```yaml
env:
- name: PASSWORD
  value: "super-secret-password"  # DON'T DO THIS!
```
✅ **Use Sealed Secrets or External Secrets**
```yaml
env:
- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: myapp-secret
      key: password
```

❌ **Different Process per Environment**
```bash
# Dev: kubectl apply
# Stage: Helm
# Prod: Manual + prayers
```
✅ **Same GitOps Process Everywhere**
```bash
# All environments: Git commit → Argo CD sync
```

## Key Takeaways

1. **Four Principles**: Declarative, Versioned, Pulled, Reconciled
2. **Git is Truth**: What's in Git should be in the cluster
3. **Pull Model**: Cluster pulls from Git, don't push to cluster
4. **Continuous Sync**: Not deploy-and-forget, continuous reconciliation
5. **Use PRs**: Leverage Git workflows for operations
6. **Test in Lower Envs**: Progressive promotion through environments

## Demo Steps

1. **Show Git History**:
   ```bash
   git log --oneline --graph
   ```

2. **Show PR-Based Change**:
   ```bash
   # Create branch
   git checkout -b update-replicas
   # Edit file
   vim k8s/deployment.yaml
   # Commit and push
   git commit -am "Scale to 5 replicas"
   git push origin update-replicas
   # Create PR (show in GitHub UI)
   ```

3. **Show Rollback**:
   ```bash
   # Revert to previous commit
   git revert HEAD
   git push
   # Show how Argo CD would sync
   ```

## What's Next?

You now understand the fundamentals of GitOps and why Argo CD is the perfect tool for implementing it. In the next section, we'll dive deep into Argo CD's architecture and core concepts before getting hands-on with our first deployment.

---

**Next Section**: [Section 2: Argo CD Architecture & Concepts](../section-02/README.md)

**Complete**: [Section 1 Quiz](../../quizzes/section-01-quiz.md) before moving on!
