# Lecture 4.1: Create a Simple App Repository

## Introduction

In GitOps, everything starts with Git. Before we can deploy with Argo CD, we need a Git repository containing our Kubernetes manifests. In this lecture, you'll create a simple application repository that Argo CD can deploy.

## The GitOps Repository Pattern

### What Goes in a GitOps Repository?

A GitOps repository contains:
- **Kubernetes manifests** (YAML files)
- **Configuration files** (ConfigMaps, Secrets metadata)
- **Application definitions** (if using App-of-Apps pattern)

**What NOT to include**:
- Application source code (goes in separate repo)
- Compiled binaries
- Secrets (use secret management tools)
- Large binary files

### Repository Structures

**Option 1: Simple Structure** (we'll use this)
```
my-app/
├── README.md
├── deployment.yaml
├── service.yaml
└── configmap.yaml
```

**Option 2: Environment-Based**
```
my-app/
├── dev/
│   ├── deployment.yaml
│   └── service.yaml
├── staging/
│   ├── deployment.yaml
│   └── service.yaml
└── production/
    ├── deployment.yaml
    └── service.yaml
```

**Option 3: Kustomize-Based** (covered in Section 6)
```
my-app/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── dev/
    ├── staging/
    └── production/
```

## Creating Your Git Repository

### Option 1: Use GitHub

1. Go to https://github.com
2. Click "New repository"
3. Name it: `argocd-demo-app`
4. Make it **Public** (or Private with credentials)
5. Initialize with README
6. Click "Create repository"

### Option 2: Use GitLab/Bitbucket

Similar process - create a new repository named `argocd-demo-app`.

### Option 3: Local Git Server (Advanced)

For this course, we recommend GitHub for simplicity.

## Creating a Simple Application

We'll create a basic nginx deployment.

### Step 1: Clone Your Repository

```bash
# Clone your repository
git clone https://github.com/<your-username>/argocd-demo-app.git
cd argocd-demo-app
```

### Step 2: Create Deployment

Create `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
  labels:
    app: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
```

### Step 3: Create Service

Create `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-demo
  labels:
    app: nginx-demo
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: nginx-demo
```

### Step 4: Create ConfigMap (Optional)

Create `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-demo-config
data:
  message: "Hello from Argo CD!"
  environment: "development"
```

### Step 5: Create README

Create or update `README.md`:

```markdown
# Argo CD Demo Application

This is a simple nginx application managed by Argo CD.

## Resources

- **Deployment**: nginx-demo (2 replicas)
- **Service**: nginx-demo (ClusterIP)
- **ConfigMap**: nginx-demo-config

## Deploying

This application is deployed using Argo CD. See main course repository for instructions.

## Testing Locally

```bash
# Apply manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f configmap.yaml

# Check deployment
kubectl get pods -l app=nginx-demo

# Port forward to test
kubectl port-forward svc/nginx-demo 8081:80

# Access at http://localhost:8081
```

## Updating

Update the manifests in this repository and Argo CD will sync the changes.
```

### Step 6: Commit and Push

```bash
# Add all files
git add .

# Commit
git commit -m "Initial commit: Add nginx demo application"

# Push to GitHub
git push origin main
```

## Verifying Your Repository

### Check on GitHub

1. Go to your repository URL
2. Verify all files are present:
   - deployment.yaml
   - service.yaml
   - configmap.yaml
   - README.md

### Test Manifests Locally

```bash
# Validate YAML syntax
kubectl apply --dry-run=client -f deployment.yaml
kubectl apply --dry-run=client -f service.yaml
kubectl apply --dry-run=client -f configmap.yaml

# Expected output for each:
# deployment.apps/nginx-demo created (dry run)
# service/nginx-demo created (dry run)
# configmap/nginx-demo-config created (dry run)
```

### Apply Manually (Optional Test)

```bash
# Create test namespace
kubectl create namespace test-app

# Apply manifests
kubectl apply -f deployment.yaml -n test-app
kubectl apply -f service.yaml -n test-app
kubectl apply -f configmap.yaml -n test-app

# Check resources
kubectl get all -n test-app

# Clean up
kubectl delete namespace test-app
```

## Common Repository Patterns

### Pattern 1: App-Specific Repository

```
Each app has its own repository:
- frontend-app (repo)
- backend-app (repo)
- database-app (repo)
```

**Pros**: Clear ownership, independent versioning  
**Cons**: Many repositories to manage

### Pattern 2: Monorepo

```
All apps in one repository:
apps/
├── frontend/
├── backend/
└── database/
```

**Pros**: Easy to manage, atomic changes  
**Cons**: Can become large, all teams have access

### Pattern 3: Environment Repositories

```
Separate repos per environment:
- apps-dev (repo)
- apps-staging (repo)
- apps-production (repo)
```

**Pros**: Clear separation, different access controls  
**Cons**: Changes must be promoted across repos

## Best Practices

✅ **Keep manifests simple** initially  
✅ **Use labels consistently** for resource management  
✅ **Include README** with instructions  
✅ **Test manifests** before committing  
✅ **Use meaningful commit messages**  
✅ **Avoid secrets** in Git (use sealed secrets later)

## Troubleshooting

### Can't push to repository

**Error**: `Permission denied (publickey)`

**Solution**:
```bash
# Set up SSH key or use HTTPS with token
# For HTTPS:
git remote set-url origin https://<token>@github.com/<user>/repo.git
```

### YAML syntax errors

**Error**: `error: yaml: line X: could not find expected ':'`

**Solution**:
```bash
# Use yamllint (optional tool)
yamllint deployment.yaml

# Or check indentation carefully
# YAML is sensitive to spaces!
```

### Manifest validation fails

**Error**: `error: error validating "deployment.yaml"`

**Solution**:
```bash
# Check resource definition
kubectl explain deployment.spec

# Verify API version
kubectl api-resources | grep Deployment
```

## Key Takeaways

✅ **Git is the source of truth** for GitOps  
✅ **Simple structure** works best initially  
✅ **Valid YAML** is essential  
✅ **Test locally** before deploying with Argo CD  
✅ **Repository ready** for Argo CD deployment

## Common Questions

**Q: Can I use a private repository?**  
A: Yes! You'll need to add credentials to Argo CD.

**Q: Do I need separate repositories for each app?**  
A: No, you can use one repository with directories for each app.

**Q: Can I use GitLab/Bitbucket instead of GitHub?**  
A: Yes! Argo CD supports all Git providers.

**Q: Should I commit secrets to Git?**  
A: Never! Use Sealed Secrets or External Secrets (covered later).

**Q: What branch should I use?**  
A: `main` or `master` is fine. You can use any branch.

## Next Steps

Now that you have a Git repository with Kubernetes manifests, continue to [Lecture 4.2: Create Your First Argo CD Application](./lecture-4.2.md) to deploy it with Argo CD!
