# Lecture 1.2: What is GitOps (Real-World View)

**Duration**: 6 minutes

## Instructor Script

"Let's talk about GitOps from a real-world perspective, not just the marketing buzzwords.

Imagine you're managing deployments across multiple Kubernetes clusters. Traditionally, someone runs kubectl apply, maybe from CI/CD, maybe manually. But here's the problem: after that deploy, how do you know what's actually running in production? Did someone make a manual hotfix? Did a pod restart with different environment variables? Is production actually what you think it is?

GitOps solves this by flipping the model. Instead of pushing changes to your cluster, the cluster pulls changes from Git. Git becomes your single source of truth. Want to know what's in production? Look at the production branch in Git. Want to make a change? Create a PR, get it reviewed, merge it, and the change automatically appears in your cluster.

This is powerful because Git already gives you everything you need: version control, audit logs, code review workflows, and rollback capabilities. You're not learning a new system – you're leveraging what you already know.

That's GitOps. It's elegant, it's auditable, and it makes operations feel like software development."

## What is GitOps?

GitOps is an operational framework that takes DevOps best practices used for application development (version control, collaboration, compliance, CI/CD) and applies them to infrastructure automation.

### Core Concept

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│              │         │              │         │              │
│  Git Repo    │────────▶│   GitOps     │────────▶│  Kubernetes  │
│  (Desired)   │         │   Operator   │         │   Cluster    │
│              │         │  (Argo CD)   │         │   (Actual)   │
└──────────────┘         └──────────────┘         └──────────────┘
       │                        │                         │
       │                        │                         │
       └────────────────────────┴─────────────────────────┘
                    Continuous Reconciliation
```

### The GitOps Principles

1. **Declarative**: Everything is described declaratively (YAML manifests)
2. **Versioned**: All configurations are stored in Git
3. **Pulled Automatically**: Changes are pulled from Git, not pushed
4. **Continuously Reconciled**: System constantly syncs desired vs actual state

## Traditional vs GitOps Deployment

### Traditional CD Pipeline

```bash
# Developer makes change
git commit -m "Update image to v2.0"
git push

# CI/CD pipeline runs
CI builds image → CI pushes to registry → CI runs kubectl apply

# Problems:
# - Who has kubectl access to production?
# - What if someone makes manual changes?
# - How do you audit who changed what?
# - How do you rollback?
```

**Issues**:
- Requires cluster credentials in CI/CD
- No single source of truth
- Manual changes create drift
- Difficult to audit and rollback
- Different process for different environments

### GitOps Deployment

```bash
# Developer makes change
git commit -m "Update image to v2.0"
git push

# Argo CD detects change
Argo CD polls Git → Detects difference → Applies to cluster

# Benefits:
# - No cluster credentials needed by developers or CI
# - Git is the source of truth
# - All changes are audited in Git
# - Rollback is just a git revert
```

**Advantages**:
- Git as single source of truth
- Automatic drift detection and correction
- Built-in audit trail
- Easy rollbacks (git revert)
- Consistent process across all environments
- Better security (pull model)

## Real-World Example

### Scenario: Deploy a New Feature

**Without GitOps**:
```bash
# Developer builds and deploys
docker build -t myapp:v2.0 .
docker push myapp:v2.0
kubectl set image deployment/myapp myapp=myapp:v2.0

# Problem: No record of who deployed what when
# Problem: What if kubectl config points to wrong cluster?
# Problem: How do other team members know about this change?
```

**With GitOps**:
```bash
# Developer updates Git
# File: k8s/deployment.yaml
# Change: image: myapp:v1.0 → image: myapp:v2.0
git commit -m "Deploy myapp v2.0"
git push

# Create PR for review
gh pr create --title "Deploy myapp v2.0"

# After approval and merge:
# Argo CD automatically syncs the change
# All changes are audited in Git history
# Team can see exactly what changed and when
```

## Why GitOps Matters

### 1. **Audit Trail**
Every change is a Git commit with:
- Who made the change
- When it was made
- Why it was made (commit message)
- What exactly changed (diff)

### 2. **Disaster Recovery**
Your entire infrastructure is in Git. If a cluster is lost:
```bash
# Recreate cluster
kind create cluster --name production

# Restore all applications
kubectl apply -f argo-cd-app-of-apps.yaml

# Everything is restored from Git
```

### 3. **Consistency**
Same process for dev, stage, and production. Same tools, same workflows, same approval processes.

### 4. **Developer Experience**
Developers use familiar tools (Git, PRs) instead of learning kubectl, helm, and cluster access patterns.

### 5. **Security**
No need to distribute cluster credentials. Argo CD pulls changes; developers never need direct cluster access.

## GitOps Anti-Patterns (What NOT to Do)

❌ **Storing secrets in plain text in Git**
✅ Use Sealed Secrets, External Secrets Operator, or SOPS

❌ **Manual kubectl commands on production**
✅ All changes through Git commits

❌ **Updating Git after deploying**
✅ Update Git first, let Argo CD deploy

❌ **Different processes for different environments**
✅ Same GitOps workflow for all environments

## Key Takeaways

1. **Git is the Source of Truth**: What's in Git is what should be in the cluster
2. **Pull, Don't Push**: Cluster pulls changes from Git automatically
3. **Declarative over Imperative**: Describe desired state, not steps to get there
4. **Continuous Reconciliation**: System constantly ensures Git matches cluster
5. **Better Security**: No need to share cluster credentials widely

## Demo Steps

Open a browser and visit these resources:
1. Show [GitOps Working Group](https://opengitops.dev/) - Official GitOps principles
2. Show a sample Git repository structure for GitOps
3. Diagram the flow: Git → Argo CD → Kubernetes

## What's Next?

Now that you understand GitOps, let's explore why Argo CD is the leading tool for implementing GitOps in Kubernetes environments.

---

**Next Lecture**: [1.3 Why Argo CD vs Traditional CD](./lecture-1.3.md)
