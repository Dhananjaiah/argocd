# Lecture 6.1: Base and Overlays

## Introduction

Kustomize enables managing multiple environments without duplicating YAML. This lecture introduces the base and overlay pattern.

## Kustomize Concepts

### Base

**What it is**: Common configuration shared across all environments

**Location**: \`base/\` directory

**Contains**:
- Deployment
- Service
- ConfigMap
- Other common resources

### Overlay

**What it is**: Environment-specific modifications

**Location**: \`overlays/dev/\`, \`overlays/prod/\`, etc.

**Contains**:
- kustomization.yaml (references base)
- Patches (changes to base)
- Additional resources

## Directory Structure

\`\`\`
my-app/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
\`\`\`

## Creating Base

\`\`\`yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
\`\`\`

\`\`\`yaml
# base/deployment.yaml
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
\`\`\`

## Creating Overlay

\`\`\`yaml
# overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
  - ../../base

namePrefix: dev-

commonLabels:
  environment: dev

replicas:
  - name: myapp
    count: 2
\`\`\`

## Testing Kustomize

\`\`\`bash
# Build and view output
kustomize build overlays/dev

# Test with kubectl
kustomize build overlays/dev | kubectl apply --dry-run=client -f -
\`\`\`

## Using with Argo CD

\`\`\`bash
argocd app create myapp-dev   --repo https://github.com/user/repo   --path overlays/dev   --dest-server https://kubernetes.default.svc   --dest-namespace dev
\`\`\`

## Key Takeaways

✅ Base = common configuration
✅ Overlay = environment-specific
✅ DRY principle (Don't Repeat Yourself)
✅ Easy to manage multiple environments
✅ Built into kubectl and Argo CD

## Next Steps

Continue to [Lecture 6.2: Dev/Stage/Prod Overlays](./lecture-6.2.md)!)

