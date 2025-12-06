# Lecture 2.2: Apps, Projects, Repos

## Introduction

Argo CD has three core resource types that you'll work with constantly: Applications, Projects, and Repositories. Understanding the relationship between these resources is essential for organizing your GitOps workflow effectively.

## The Three Core Resources

```
┌─────────────┐
│ Repository  │  Where your manifests live (Git)
└──────┬──────┘
       │
       │ referenced by
       │
┌──────▼──────┐
│   Project   │  Grouping and access control
└──────┬──────┘
       │
       │ contains
       │
┌──────▼──────┐
│ Application │  What to deploy and where
└─────────────┘
```

## Applications

### What is an Application?

An Application is the primary resource in Argo CD. It defines:
- **What** to deploy (source repository and path)
- **Where** to deploy (destination cluster and namespace)
- **How** to deploy (sync policy, parameters)

Think of it as a deployment manifest for GitOps.

### Application Structure

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  # WHAT to deploy
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps
    targetRevision: HEAD
    path: guestbook
  
  # WHERE to deploy
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
  
  # HOW to deploy
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Key Application Components

#### Source

Defines where the manifests come from:

```yaml
source:
  repoURL: https://github.com/org/repo       # Git repository URL
  targetRevision: main                       # Branch, tag, or commit SHA
  path: kubernetes/manifests                 # Path within repo
```

For Helm:
```yaml
source:
  repoURL: https://github.com/org/repo
  path: charts/myapp
  helm:
    values: |
      replicas: 3
      image:
        tag: v1.0.0
```

For Kustomize:
```yaml
source:
  repoURL: https://github.com/org/repo
  path: overlays/production
  kustomize:
    namePrefix: prod-
    images:
      - myapp:v1.0.0
```

#### Destination

Defines where to deploy:

```yaml
destination:
  server: https://kubernetes.default.svc    # Kubernetes API server
  namespace: production                     # Target namespace
```

You can also use cluster name instead of server URL:
```yaml
destination:
  name: prod-cluster
  namespace: production
```

#### Sync Policy

Controls how and when to sync:

```yaml
syncPolicy:
  automated:                 # Enable auto-sync
    prune: true              # Delete resources not in Git
    selfHeal: true           # Revert manual changes
  syncOptions:
    - CreateNamespace=true   # Create namespace if missing
  retry:
    limit: 5
    backoff:
      duration: 5s
```

### Application Status

Every Application has a status with:

**Sync Status**:
- `Synced`: Live state matches Git
- `OutOfSync`: Differences detected
- `Unknown`: Cannot determine status

**Health Status**:
- `Healthy`: All resources running correctly
- `Progressing`: Resources being created/updated
- `Degraded`: Resources have issues
- `Suspended`: Intentionally suspended
- `Missing`: Resources don't exist

**Operation State**:
- `Running`: Sync operation in progress
- `Failed`: Sync operation failed
- `Succeeded`: Sync operation succeeded

### Creating Applications

**Via CLI**:
```bash
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

**Via YAML**:
```bash
kubectl apply -f application.yaml
```

**Via UI**:
1. Click "New App"
2. Fill in form
3. Click "Create"

## Projects

### What is a Project?

A Project (AppProject) is a logical grouping of Applications with:
- Access restrictions (which repos, clusters, namespaces)
- Resource restrictions (which Kubernetes resources)
- RBAC policies
- Sync windows

Projects enable multi-tenancy in Argo CD.

### Default Project

Every Argo CD installation has a `default` project:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: default
  namespace: argocd
spec:
  # Allow deploying from any repo
  sourceRepos:
    - '*'
  
  # Allow deploying to any cluster/namespace
  destinations:
    - namespace: '*'
      server: '*'
  
  # Allow all resources
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
```

### Custom Project Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-a
  namespace: argocd
spec:
  description: Team A's applications
  
  # Restrict source repositories
  sourceRepos:
    - https://github.com/org/team-a-*
    - https://github.com/org/shared-charts
  
  # Restrict deployment destinations
  destinations:
    - namespace: team-a-*
      server: https://kubernetes.default.svc
    - namespace: team-a-*
      server: https://prod-cluster.example.com
  
  # Allow specific cluster-scoped resources
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
    - group: 'rbac.authorization.k8s.io'
      kind: ClusterRole
  
  # Deny specific resources
  namespaceResourceBlacklist:
    - group: ''
      kind: ResourceQuota
    - group: ''
      kind: LimitRange
  
  # RBAC roles
  roles:
    - name: developer
      description: Developer access
      policies:
        - p, proj:team-a:developer, applications, get, team-a/*, allow
        - p, proj:team-a:developer, applications, sync, team-a/*, allow
      groups:
        - team-a-devs
```

### Project Benefits

1. **Multi-Tenancy**: Isolate teams and applications
2. **Security**: Restrict what can be deployed where
3. **Organization**: Group related applications
4. **RBAC**: Fine-grained access control per project

### Creating Projects

**Via CLI**:
```bash
argocd proj create team-a \
  --description "Team A's project" \
  --src https://github.com/org/team-a-* \
  --dest https://kubernetes.default.svc,team-a-*
```

**Via YAML**:
```bash
kubectl apply -f project.yaml
```

## Repositories

### What is a Repository?

A Repository (repo) connection defines how Argo CD accesses your Git repositories or Helm chart repositories.

### Git Repository

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: private-repo
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: https://github.com/org/private-repo
  password: ghp_xxxxxxxxxxxxx
  username: git
```

### Repository Types

#### Public Git Repository

```bash
argocd repo add https://github.com/org/public-repo
```

No credentials needed.

#### Private Git Repository (HTTPS)

```bash
argocd repo add https://github.com/org/private-repo \
  --username git \
  --password ghp_token123
```

#### Private Git Repository (SSH)

```bash
argocd repo add git@github.com:org/private-repo.git \
  --ssh-private-key-path ~/.ssh/id_rsa
```

#### Helm Chart Repository

```bash
argocd repo add https://charts.helm.sh/stable \
  --type helm \
  --name stable
```

### Repository Credentials

Argo CD supports multiple authentication methods:

1. **HTTPS with Token**: GitHub/GitLab personal access token
2. **HTTPS with Username/Password**: Basic auth
3. **SSH**: Private key authentication
4. **GitHub App**: GitHub App installation
5. **Google Cloud Source**: GCP credentials

### Repository Templates

For organizations with many repos, use credential templates:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: github-org-creds
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repo-creds
stringData:
  type: git
  url: https://github.com/myorg
  password: ghp_xxxxxxxxxxxxx
  username: git
```

Now any repo matching `https://github.com/myorg/*` uses these credentials.

## How They Work Together

### Example Workflow

```
1. Add Repository to Argo CD
   └─> Credentials stored securely

2. Create Project (Optional but recommended)
   └─> Define access boundaries

3. Create Application
   ├─> References Repository (source)
   ├─> Belongs to Project
   └─> Specifies Destination

4. Application Controller
   ├─> Uses Repository credentials to access Git
   ├─> Enforces Project restrictions
   └─> Deploys to specified Destination
```

### Real-World Example

**Scenario**: Team A manages microservices in production

**Step 1: Add Repository**
```bash
argocd repo add https://github.com/company/team-a-apps \
  --username git \
  --password $GITHUB_TOKEN
```

**Step 2: Create Project**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-a-production
spec:
  sourceRepos:
    - https://github.com/company/team-a-apps
  destinations:
    - namespace: team-a-*
      server: https://prod-cluster
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
```

**Step 3: Create Applications**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: frontend-prod
spec:
  project: team-a-production
  source:
    repoURL: https://github.com/company/team-a-apps
    path: frontend/prod
  destination:
    server: https://prod-cluster
    namespace: team-a-frontend
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: backend-prod
spec:
  project: team-a-production
  source:
    repoURL: https://github.com/company/team-a-apps
    path: backend/prod
  destination:
    server: https://prod-cluster
    namespace: team-a-backend
```

## Best Practices

### Applications

✅ Use descriptive names: `frontend-prod`, not `app1`  
✅ Keep sync policies consistent per environment  
✅ Always specify a project (don't rely on `default`)  
✅ Use targetRevision for production (specific tag/commit)

### Projects

✅ Create projects per team or environment  
✅ Restrict source repos to prevent unauthorized deployments  
✅ Limit destinations to prevent cross-team interference  
✅ Use RBAC roles for access control

### Repositories

✅ Use credential templates for organization-wide access  
✅ Prefer SSH keys or tokens over passwords  
✅ Rotate credentials regularly  
✅ Use least-privilege tokens (read-only for Git)

## Key Takeaways

✅ **Application**: What, where, and how to deploy  
✅ **Project**: Grouping and access control  
✅ **Repository**: How to access Git repos  
✅ **Relationship**: Repos → Projects → Applications  
✅ **Multi-tenancy**: Use Projects to isolate teams

## Common Questions

**Q: Can an Application belong to multiple Projects?**  
A: No, each Application belongs to exactly one Project.

**Q: Can an Application use multiple repositories?**  
A: Not directly, but you can use Helm dependencies or git submodules.

**Q: Do I need to create a Project?**  
A: Not required, but recommended for organization and security.

**Q: Can I change an Application's Project?**  
A: Yes, but ensure the new Project allows the Application's source and destination.

## Next Steps

Now that you understand Apps, Projects, and Repos, continue to [Lecture 2.3: Desired vs Live State](./lecture-2.3.md) to learn about state management in Argo CD.
