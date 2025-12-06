# Section 6: Kustomize with Argo CD

## Overview

Learn how to use Kustomize with Argo CD to manage multi-environment configurations without duplication. Kustomize allows you to maintain a single base configuration and overlay environment-specific changes.

## Learning Objectives

By the end of this section, you will:
- Understand Kustomize concepts (base, overlays, patches)
- Create base manifests and environment overlays
- Deploy to dev/stage/prod with different configurations
- Troubleshoot Kustomize build issues
- Apply DRY principles to Kubernetes configurations

## Lectures

1. [6.1 Base and Overlays](./lecture-6.1.md)
2. [6.2 Dev/Stage/Prod Overlays](./lecture-6.2.md)
3. [6.3 Parameter Changes Per Environment](./lecture-6.3.md)
4. [6.4 Troubleshooting Kustomize Builds](./lecture-6.4.md)

## Hands-On Labs

- **[Lab 4: Using Kustomize with Argo CD](../../labs/lab-04.md)** - Basic Kustomize usage
- **[Lab 5: Multi-Environment with Kustomize](../../labs/lab-05.md)** - Advanced patterns

## Key Takeaways

- **Base**: Common configuration shared across environments
- **Overlays**: Environment-specific modifications
- **Patches**: Strategic merge or JSON patches for changes
- **DRY Principle**: Don't Repeat Yourself - single source of truth
- **Kustomize is built into kubectl and Argo CD**

## Kustomize Structure

```
kustomize-app/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patches/
    │       ├── replica-patch.yaml
    │       └── resource-patch.yaml
    ├── staging/
    │   ├── kustomization.yaml
    │   └── patches/
    └── production/
        ├── kustomization.yaml
        └── patches/
```

## Base Example

**base/kustomization.yaml**:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml

commonLabels:
  app: myapp
  managed-by: kustomize
```

**base/deployment.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
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
      - name: app
        image: myapp:latest
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
```

## Overlay Examples

### Development Overlay

**overlays/dev/kustomization.yaml**:
```yaml
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
  - name: myapp
    newTag: dev-latest
```

### Production Overlay

**overlays/production/kustomization.yaml**:
```yaml
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

images:
  - name: myapp
    newTag: v1.2.3

patchesStrategicMerge:
  - resource-patch.yaml

configMapGenerator:
  - name: app-config
    literals:
      - ENV=production
      - LOG_LEVEL=info
```

**overlays/production/resource-patch.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: app
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 1Gi
```

## Kustomize Features

### 1. Name Prefixes/Suffixes

```yaml
namePrefix: dev-
nameSuffix: -v2
```

### 2. Common Labels

```yaml
commonLabels:
  app: myapp
  environment: dev
  team: platform
```

### 3. Replica Count

```yaml
replicas:
  - name: myapp
    count: 3
```

### 4. Image Tags

```yaml
images:
  - name: nginx
    newName: nginx
    newTag: 1.25
```

### 5. ConfigMap/Secret Generators

```yaml
configMapGenerator:
  - name: app-config
    literals:
      - KEY=value
    files:
      - config.properties

secretGenerator:
  - name: app-secret
    literals:
      - PASSWORD=secret
    type: Opaque
```

### 6. Strategic Merge Patches

```yaml
patchesStrategicMerge:
  - replica-patch.yaml
  - resource-patch.yaml
```

### 7. JSON Patches

```yaml
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

## Using with Argo CD

### Via CLI

```bash
argocd app create myapp-dev \
  --repo https://github.com/org/repo.git \
  --path overlays/dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace dev
```

### Via YAML

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/repo.git
    targetRevision: main
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: false
      selfHeal: true
```

## Testing Kustomize

```bash
# Build and view output
kustomize build overlays/dev

# Test apply (dry-run)
kustomize build overlays/dev | kubectl apply --dry-run=client -f -

# Apply to cluster
kustomize build overlays/dev | kubectl apply -f -

# Or use kubectl directly
kubectl apply -k overlays/dev
```

## Best Practices

### 1. Keep Base Generic

Base should have sensible defaults that work everywhere.

### 2. Minimal Overlays

Only include what changes, not entire manifests.

### 3. Test Locally

Always test with `kustomize build` before committing.

### 4. Consistent Structure

Use same structure across all projects.

### 5. Version Control

Commit base and all overlays.

### 6. Document Differences

Add README explaining what each overlay changes.

## Common Patterns

### Pattern 1: Environment-Specific Resources

**Different resource limits per environment**:
```yaml
# dev: small resources
# staging: medium resources
# prod: large resources + multiple replicas
```

### Pattern 2: Feature Flags

```yaml
# Enable features only in certain environments
configMapGenerator:
  - name: features
    literals:
      - FEATURE_X=true  # Only in dev
```

### Pattern 3: External Dependencies

```yaml
# Different service endpoints per environment
# dev: dev-database.local
# prod: prod-database.com
```

## Troubleshooting

### Issue: Kustomize Build Fails

```bash
# Test locally
kustomize build overlays/dev

# Check syntax
yamllint kustomization.yaml

# Verify file paths
ls -la base/
```

### Issue: Wrong Values Applied

```bash
# View generated output
argocd app manifests myapp-dev

# Check which overlay is used
argocd app get myapp-dev | grep Path
```

### Issue: Name Conflicts

Use namePrefix or nameSuffix to avoid conflicts:
```yaml
namePrefix: dev-
```

## Next Steps

After completing this section and Labs 4-5:
- You'll master Kustomize with Argo CD
- You'll manage multiple environments efficiently
- You'll be ready to learn Helm charts
- Continue to [Section 7: Helm with Argo CD](../section-07/README.md)

## Quiz

Complete the [Section 6 Quiz](../../quizzes/section-06-quiz.md) to test your understanding.

## Additional Resources

- [Kustomize Documentation](https://kustomize.io/)
- [Kustomize Example](../../examples/kustomize-demo/)
- [Argo CD Kustomize Guide](https://argo-cd.readthedocs.io/en/stable/user-guide/kustomize/)
