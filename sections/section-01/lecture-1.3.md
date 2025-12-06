# Lecture 1.3: Why Argo CD vs Traditional CD

**Duration**: 6 minutes

## Instructor Script

"You might be wondering – we already have CI/CD tools like Jenkins, GitLab CI, GitHub Actions. Why do we need Argo CD?

The key insight is this: CI/CD tools were designed to push changes. They're great at building code, running tests, and pushing artifacts. But they weren't designed for the Kubernetes pull model. They require cluster credentials, they run imperatively, and they don't continuously reconcile state.

Argo CD was built specifically for Kubernetes and GitOps. It runs inside your cluster, watches your Git repositories, and continuously ensures your cluster matches what's in Git. It understands Kubernetes resources natively, provides a beautiful UI for visualizing your applications, and gives you features like automated sync, drift detection, and self-healing.

Think of it this way: your CI/CD tool builds and tests your code. Argo CD deploys and manages it in Kubernetes. They work together, but they have different jobs."

## Traditional CD Tools

### Examples
- Jenkins
- GitLab CI/CD
- GitHub Actions
- CircleCI
- Travis CI

### How They Work

```yaml
# .github/workflows/deploy.yml
name: Deploy to Kubernetes
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to cluster
        run: |
          kubectl apply -f k8s/
        env:
          KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

### Problems with Push-Based CD

1. **Credentials Management**
   - CI/CD needs cluster access tokens
   - Credentials stored in CI/CD secrets
   - Security risk if CI/CD is compromised

2. **No Continuous Reconciliation**
   - Deploys once and forgets
   - Manual changes create drift
   - No automatic recovery

3. **Not Kubernetes-Native**
   - Treats Kubernetes as black box
   - Uses kubectl as external tool
   - Doesn't understand resource relationships

4. **Limited Visibility**
   - No live view of cluster state
   - Difficult to see drift
   - Hard to debug deployment issues

5. **Complex Multi-Cluster**
   - Need credentials for each cluster
   - Duplicate pipeline logic
   - Hard to manage consistency

## Argo CD: The GitOps Way

### Key Differences

```
Traditional CD          │  Argo CD
──────────────────────  │  ────────────────────
Push model              │  Pull model
Runs outside cluster    │  Runs inside cluster
Needs cluster creds     │  Uses in-cluster auth
Deploy once             │  Continuous sync
No drift detection      │  Automatic drift detection
Imperative commands     │  Declarative state
```

### Architecture

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  Kubernetes Cluster                             │
│                                                 │
│  ┌────────────────────────────────────────┐    │
│  │  Argo CD                               │    │
│  │                                        │    │
│  │  ┌──────────┐      ┌──────────┐      │    │
│  │  │   API    │      │   Repo   │      │    │
│  │  │  Server  │◀────▶│  Server  │      │    │
│  │  └──────────┘      └──────────┘      │    │
│  │       ▲                  │            │    │
│  │       │                  │            │    │
│  │       │            ┌──────────┐       │    │
│  │       │            │ App      │       │    │
│  │       └────────────│Controller│       │    │
│  │                    └──────────┘       │    │
│  │                         │              │    │
│  └─────────────────────────┼──────────────┘    │
│                            │                   │
│                            ▼                   │
│            ┌───────────────────────┐           │
│            │  Kubernetes Resources │           │
│            └───────────────────────┘           │
└─────────────────────────────────────────────────┘
                       │
                       │ Polls for changes
                       ▼
              ┌─────────────────┐
              │   Git Repo      │
              └─────────────────┘
```

### Advantages of Argo CD

#### 1. **Kubernetes Native**

```bash
# Argo CD understands Kubernetes resources
argocd app get myapp

# Shows:
# - Deployment status
# - Pod health
# - Service endpoints
# - Resource relationships (parent/child)
```

#### 2. **Continuous Reconciliation**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  syncPolicy:
    automated:
      prune: true      # Delete resources not in Git
      selfHeal: true   # Revert manual changes
```

If someone runs `kubectl edit deployment myapp`, Argo CD automatically reverts it to match Git!

#### 3. **Better Security**

```
Traditional CD:
CI/CD ──[Cluster Credentials]──▶ Kubernetes

Argo CD:
Argo CD (inside cluster) ◀──[Polls]── Git
     │
     └──▶ Kubernetes (ServiceAccount)
```

No cluster credentials leave the cluster!

#### 4. **Excellent Visibility**

```bash
# Beautiful web UI showing:
# - Application health and sync status
# - Resource tree visualization
# - Live logs and events
# - Diff between Git and cluster
# - History of syncs

# CLI also powerful:
argocd app get myapp --show-params
argocd app diff myapp
argocd app history myapp
```

#### 5. **Multi-Cluster Made Easy**

```yaml
# Register multiple clusters
argocd cluster add dev-cluster
argocd cluster add stage-cluster
argocd cluster add prod-cluster

# Deploy to any cluster
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
spec:
  destination:
    server: https://prod-cluster
```

#### 6. **Advanced Deployment Patterns**

```yaml
# Sync waves - control order
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"

# Hooks - run jobs at specific times
metadata:
  annotations:
    argocd.argoproj.io/hook: PreSync

# Custom health checks
# Resource tracking
# And more...
```

## When to Use What

### Use Traditional CI/CD For:
- ✅ Building code
- ✅ Running tests
- ✅ Building Docker images
- ✅ Pushing to image registry
- ✅ Running security scans

### Use Argo CD For:
- ✅ Deploying to Kubernetes
- ✅ Managing configurations
- ✅ Syncing Git to cluster
- ✅ Multi-environment deployments
- ✅ Drift detection and correction

## Best Practice: Use Both Together

```yaml
# .github/workflows/ci.yml
name: CI
on: [push]
jobs:
  build:
    steps:
      - name: Build and test
        run: npm test
      - name: Build image
        run: docker build -t myapp:${{ github.sha }}
      - name: Push image
        run: docker push myapp:${{ github.sha }}
      - name: Update Git with new image tag
        run: |
          sed -i "s/image: .*/image: myapp:${{ github.sha }}/" k8s/deployment.yaml
          git commit -am "Update to ${{ github.sha }}"
          git push
      # Argo CD will detect the change and deploy automatically!
```

## Real-World Comparison

### Scenario: Rollback a Broken Deployment

**Traditional CD**:
```bash
# Find the previous working version
# Update pipeline or run manual deploy
# Hope you remember the right version
# May need to rebuild images
# Takes 10-15 minutes
```

**Argo CD**:
```bash
# In UI or CLI
argocd app rollback myapp

# Or in Git
git revert HEAD
git push

# Argo CD syncs in seconds
# Takes < 1 minute
```

### Scenario: Someone Makes Manual Change

**Traditional CD**:
```bash
# kubectl edit deployment myapp
# Change made
# CD doesn't know about it
# Drift exists until next deploy
# May cause issues
```

**Argo CD**:
```bash
# kubectl edit deployment myapp
# Change made
# Argo CD detects drift immediately
# With selfHeal: true, reverts automatically
# Cluster stays in sync with Git
```

## Key Takeaways

1. **Different Jobs**: CI/CD builds, Argo CD deploys
2. **Pull vs Push**: Argo CD's pull model is more secure
3. **Kubernetes Native**: Argo CD understands K8s resources deeply
4. **Continuous Sync**: Not just deploy-and-forget
5. **Better Together**: Use CI/CD for build, Argo CD for deploy

## Demo Steps

1. **Show Traditional Deployment**:
   ```bash
   kubectl apply -f deployment.yaml
   # Make manual change
   kubectl edit deployment myapp
   # Show drift exists
   ```

2. **Show Argo CD Approach**:
   ```bash
   # (After Argo CD setup in Section 3)
   argocd app sync myapp
   # Make manual change
   kubectl edit deployment myapp
   # Show Argo CD detects and reverts
   ```

## What's Next?

Now let's dive deeper into the core principles and workflows that make GitOps with Argo CD so powerful.

---

**Next Lecture**: [1.4 Core GitOps Principles & Workflows](./lecture-1.4.md)
