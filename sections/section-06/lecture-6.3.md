# Lecture 6.3: Parameter Changes Per Environment

## Introduction

Different environments need different parameters like replicas, resources, and configurations.

## Common Parameters to Change

1. **Replicas** - Number of pods
2. **Image Tags** - Version per environment
3. **Resources** - CPU/memory limits
4. **Environment Variables** - API endpoints, feature flags
5. **ConfigMaps** - Application config per environment

## Using Kustomize Features

### 1. Replica Count

\`\`\`yaml
replicas:
  - name: myapp
    count: 3
\`\`\`

### 2. Image Tags

\`\`\`yaml
images:
  - name: myapp
    newTag: v1.2.3
\`\`\`

### 3. Name Prefix/Suffix

\`\`\`yaml
namePrefix: prod-
nameSuffix: -v2
\`\`\`

### 4. Labels

\`\`\`yaml
commonLabels:
  environment: production
  team: platform
\`\`\`

### 5. ConfigMap Generator

\`\`\`yaml
configMapGenerator:
  - name: app-config
    literals:
      - ENV=production
      - API_URL=https://api.prod.example.com
      - FEATURE_X=true
\`\`\`

### 6. Patches

**Strategic Merge Patch**:
\`\`\`yaml
patchesStrategicMerge:
  - patch.yaml
\`\`\`

**JSON Patch**:
\`\`\`yaml
patchesJson6902:
  - target:
      kind: Deployment
      name: myapp
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
\`\`\`

## Complete Example

\`\`\`yaml
# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production
bases:
  - ../../base

namePrefix: prod-

commonLabels:
  environment: production
  managed-by: kustomize

replicas:
  - name: myapp
    count: 3

images:
  - name: myapp
    newName: registry.example.com/myapp
    newTag: v1.2.3

configMapGenerator:
  - name: app-config
    literals:
      - ENV=production
      - LOG_LEVEL=info
      - API_URL=https://api.prod.example.com

patchesStrategicMerge:
  - resource-patch.yaml
\`\`\`

## Key Takeaways

✅ Many ways to customize
✅ Use appropriate method for each change
✅ ConfigMap generator for config
✅ Patches for complex changes
✅ Keep it simple initially

## Next Steps

Continue to [Lecture 6.4: Troubleshooting Kustomize Builds](./lecture-6.4.md)!)

