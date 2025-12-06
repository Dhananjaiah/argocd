# Lab 4: Using Kustomize with Argo CD

**Difficulty**: Beginner  
**Duration**: 35 minutes  
**Prerequisites**: Lab 3 completed

## Goal

Learn how to use Kustomize with Argo CD to manage Kubernetes configurations with base manifests and overlays.

## What You'll Learn

- What Kustomize is and why it's useful
- How to create base manifests
- How to create overlays for different environments
- How to use Kustomize with Argo CD

## Step 1: Understand Kustomize

Kustomize is a Kubernetes configuration management tool that uses:
- **Base**: Common configuration shared across environments
- **Overlays**: Environment-specific modifications
- **Patches**: Changes applied to base resources

### Why Kustomize?

```yaml
# Problem without Kustomize:
# - deployment-dev.yaml (copy/paste, hard to maintain)
# - deployment-staging.yaml (copy/paste, hard to maintain)
# - deployment-prod.yaml (copy/paste, hard to maintain)

# Solution with Kustomize:
# - base/deployment.yaml (single source of truth)
# - overlays/dev/patches (only differences)
# - overlays/staging/patches (only differences)
# - overlays/prod/patches (only differences)
```

## Step 2: Create Base Manifests

Let's create a simple application with base manifests.

```bash
# Create directory structure
mkdir -p ~/kustomize-demo/{base,overlays/{dev,staging,prod}}
cd ~/kustomize-demo
```

### Create Base Deployment

```bash
cat <<'EOF' > base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
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
EOF
```

### Create Base Service

```bash
cat <<'EOF' > base/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
EOF
```

### Create Base Kustomization

```bash
cat <<'EOF' > base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml

commonLabels:
  app: myapp
  managed-by: kustomize
EOF
```

## Step 3: Create Dev Overlay

```bash
cat <<'EOF' > overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: dev

bases:
  - ../../base

namePrefix: dev-

commonLabels:
  environment: dev

replicas:
  - name: myapp
    count: 1

images:
  - name: nginx
    newTag: 1.25-alpine
EOF
```

## Step 4: Create Staging Overlay

```bash
cat <<'EOF' > overlays/staging/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: staging

bases:
  - ../../base

namePrefix: staging-

commonLabels:
  environment: staging

replicas:
  - name: myapp
    count: 2

# Resource patches for staging
patchesStrategicMerge:
  - replica-patch.yaml
EOF
```

```bash
cat <<'EOF' > overlays/staging/replica-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: nginx
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
EOF
```

## Step 5: Create Production Overlay

```bash
cat <<'EOF' > overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production

bases:
  - ../../base

namePrefix: prod-

commonLabels:
  environment: production

replicas:
  - name: myapp
    count: 3

# Production patches
patchesStrategicMerge:
  - replica-patch.yaml
  - resource-patch.yaml

# Production-specific config
configMapGenerator:
  - name: app-config
    literals:
      - ENV=production
      - LOG_LEVEL=info
EOF
```

```bash
cat <<'EOF' > overlays/prod/replica-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
EOF
```

```bash
cat <<'EOF' > overlays/prod/resource-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: nginx
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 1Gi
EOF
```

## Step 6: Test Kustomize Locally

### Build and View Dev

```bash
kustomize build overlays/dev
```

### Build and View Staging

```bash
kustomize build overlays/staging
```

### Build and View Production

```bash
kustomize build overlays/prod
```

**Notice the differences**:
- Different namespaces
- Different replica counts
- Different resource limits
- Different name prefixes
- Different labels

## Step 7: Push to Git

```bash
# Initialize git repo
git init
git add .
git commit -m "Initial kustomize structure"

# Push to GitHub (create repo first)
git remote add origin https://github.com/YOUR_USERNAME/kustomize-demo.git
git branch -M main
git push -u origin main
```

## Step 8: Deploy Dev with Argo CD

### Create Namespaces

```bash
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace production
```

### Create Argo CD Application for Dev

```bash
argocd app create myapp-dev \
  --repo https://github.com/YOUR_USERNAME/kustomize-demo.git \
  --path overlays/dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace dev \
  --sync-policy automated \
  --self-heal \
  --auto-prune
```

**Or using YAML**:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/YOUR_USERNAME/kustomize-demo.git
    targetRevision: main
    path: overlays/dev
  
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF
```

### Sync and Verify

```bash
argocd app wait myapp-dev --health

kubectl get all -n dev
```

**Expected Output**:
```
NAME                             READY   STATUS    RESTARTS   AGE
pod/dev-myapp-xxx                1/1     Running   0          1m

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
service/dev-myapp       ClusterIP   10.96.xxx.xxx   <none>        80/TCP

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dev-myapp   1/1     1            1           1m
```

## Step 9: Deploy Staging

```bash
argocd app create myapp-staging \
  --repo https://github.com/YOUR_USERNAME/kustomize-demo.git \
  --path overlays/staging \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace staging \
  --sync-policy automated
```

```bash
argocd app wait myapp-staging --health

kubectl get deployment -n staging
```

Should show **2 replicas** (vs 1 in dev).

## Step 10: Deploy Production

```bash
argocd app create myapp-prod \
  --repo https://github.com/YOUR_USERNAME/kustomize-demo.git \
  --path overlays/prod \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production
  # Note: NO --sync-policy automated for prod!
```

```bash
# Manual sync required for production
argocd app sync myapp-prod

argocd app wait myapp-prod --health

kubectl get deployment -n production
```

Should show **3 replicas**.

## Step 11: Make a Change

Let's change the base image and see it propagate.

```bash
cd ~/kustomize-demo
```

### Edit Base Deployment

```bash
sed -i 's/nginx:1.25/nginx:1.26/g' base/deployment.yaml
```

### Commit and Push

```bash
git add base/deployment.yaml
git commit -m "Update nginx to 1.26"
git push
```

### Watch Argo CD

```bash
# Dev auto-syncs
watch kubectl get deployment dev-myapp -n dev -o jsonpath='{.spec.template.spec.containers[0].image}'

# Staging auto-syncs
watch kubectl get deployment staging-myapp -n staging -o jsonpath='{.spec.template.spec.containers[0].image}'

# Production requires manual sync
argocd app sync myapp-prod
```

**All environments now use nginx:1.26!**

## Verification Commands

### Check All Applications

```bash
argocd app list | grep myapp
```

### Check Specific Environment

```bash
kubectl get all -n dev -l app=myapp
kubectl get all -n staging -l app=myapp
kubectl get all -n production -l app=myapp
```

### View Generated Manifests

```bash
# What Argo CD sees for dev
argocd app manifests myapp-dev

# What Argo CD sees for prod
argocd app manifests myapp-prod
```

### Compare Environments

```bash
# Dev replicas
kubectl get deployment dev-myapp -n dev -o jsonpath='{.spec.replicas}'

# Staging replicas
kubectl get deployment staging-myapp -n staging -o jsonpath='{.spec.replicas}'

# Prod replicas
kubectl get deployment prod-myapp -n production -o jsonpath='{.spec.replicas}'
```

## Troubleshooting

### Issue: Kustomize Build Fails

**Check syntax**:
```bash
kustomize build overlays/dev
```

**Common errors**:
- Wrong indentation in YAML
- Incorrect paths in bases
- Missing resources

**Fix**: Validate YAML syntax online or use `yamllint`.

### Issue: Argo CD Can't Find Path

**Verify path exists in repo**:
```bash
git ls-tree -r main --name-only | grep overlays
```

**Check Argo CD app definition**:
```bash
argocd app get myapp-dev | grep Path
```

### Issue: Changes Not Appearing

**Force refresh**:
```bash
argocd app get myapp-dev --refresh --hard-refresh
```

**Check Git commit is pushed**:
```bash
git log --oneline
```

### Issue: Multiple Resources with Same Name

Kustomize requires unique names. Use `namePrefix` or `nameSuffix`:

```yaml
namePrefix: dev-
# or
nameSuffix: -dev
```

## Best Practices

### 1. **Keep Base Generic**

```yaml
# base/ should have sensible defaults
# Don't put environment-specific values in base
```

### 2. **Use Overlays for Differences**

```yaml
# Only include what changes per environment
# Don't duplicate the entire manifest
```

### 3. **Test Locally First**

```bash
# Always test before committing
kustomize build overlays/dev | kubectl apply --dry-run=client -f -
```

### 4. **Version Control Everything**

```bash
# Commit base and all overlays
git add base/ overlays/
git commit -m "Update configurations"
```

### 5. **Use Strategic Merge Patches**

```yaml
# For simple changes (replicas, images)
patchesStrategicMerge:
  - replica-patch.yaml
```

### 6. **Use JSON Patches for Complex Changes**

```yaml
# For complex changes
patchesJson6902:
  - target:
      group: apps
      version: v1
      kind: Deployment
      name: myapp
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 3
```

## Cleanup

```bash
# Delete applications
argocd app delete myapp-dev --yes
argocd app delete myapp-staging --yes
argocd app delete myapp-prod --yes

# Delete namespaces
kubectl delete namespace dev
kubectl delete namespace staging
kubectl delete namespace production
```

## What You've Accomplished

✅ Created base Kubernetes manifests  
✅ Created environment-specific overlays  
✅ Used Kustomize to generate configurations  
✅ Deployed to multiple environments with Argo CD  
✅ Made changes that propagate across environments  
✅ Understood DRY principle with Kustomize  

## Key Concepts

1. **Base**: Common configuration shared by all environments
2. **Overlay**: Environment-specific modifications
3. **Patch**: Changes applied to base resources
4. **DRY**: Don't Repeat Yourself - maintain one source of truth
5. **Kustomize Build**: Generates final manifests from base + overlays

## Next Steps

Continue to:
- **[Lab 5: Multi-Environment with Kustomize](./lab-05.md)** - Advanced multi-environment patterns
- **[Section 6: Kustomize with Argo CD](../sections/section-06/README.md)** - Deep dive into Kustomize

## Quick Reference

```bash
# Test kustomize locally
kustomize build overlays/<env>

# Create app with kustomize
argocd app create <name> \
  --repo <url> \
  --path overlays/<env> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <namespace>

# View generated manifests
argocd app manifests <name>
```

---

**Excellent!** You now know how to use Kustomize with Argo CD for multi-environment deployments!
