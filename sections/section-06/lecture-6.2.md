# Lecture 6.2: Dev/Stage/Prod Overlays
Parameter Changes Per Environment

## Introduction

Let's create a complete multi-environment setup with dev, staging, and production overlays.

## Environment Differences

| Aspect | Dev | Staging | Production |
|--------|-----|---------|------------|
| Replicas | 1 | 2 | 3 |
| Resources | Small | Medium | Large |
| Image Tag | latest | stable | v1.0.0 |
| Namespace | dev | staging | production |

## Creating Overlays

### Development

\`\`\`yaml
# overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: dev
bases:
  - ../../base

namePrefix: dev-

replicas:
  - name: myapp
    count: 1

images:
  - name: myapp
    newTag: latest
\`\`\`

### Staging

\`\`\`yaml
# overlays/staging/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: staging
bases:
  - ../../base

namePrefix: staging-

replicas:
  - name: myapp
    count: 2

images:
  - name: myapp
    newTag: stable
\`\`\`

### Production

\`\`\`yaml
# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production
bases:
  - ../../base

namePrefix: prod-

replicas:
  - name: myapp
    count: 3

images:
  - name: myapp
    newTag: v1.0.0

patchesStrategicMerge:
  - resource-patch.yaml
\`\`\`

\`\`\`yaml
# overlays/production/resource-patch.yaml
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
\`\`\`

## Creating Argo CD Applications

\`\`\`bash
# Dev
argocd app create myapp-dev   --repo https://github.com/user/repo   --path overlays/dev   --dest-namespace dev

# Staging  
argocd app create myapp-staging   --repo https://github.com/user/repo   --path overlays/staging   --dest-namespace staging

# Production
argocd app create myapp-prod   --repo https://github.com/user/repo   --path overlays/production   --dest-namespace production
\`\`\`

## Key Takeaways

✅ One base, multiple overlays
✅ Each environment is independent
✅ Easy to see environment differences
✅ Changes to base affect all environments
✅ Overlay-specific changes stay isolated

## Next Steps

Continue to [Lecture 6.3: Parameter Changes Per Environment](./lecture-6.3.md)!)

