# Lecture 6.4: Troubleshooting Kustomize Builds

## Introduction

Kustomize can be tricky. This lecture covers common issues and how to fix them.

## Common Issues

### 1. Build Fails

**Error**: \`Error: accumulating resources: ...path not found\`

**Cause**: Wrong path in bases/resources

**Fix**:
\`\`\`bash
# Verify path is correct
ls ../../base

# Check kustomization.yaml paths
cat overlays/dev/kustomization.yaml
\`\`\`

### 2. Resource Not Found

**Error**: \`couldn't find resource deployment.yaml\`

**Cause**: Resource file doesn't exist

**Fix**: Check spelling and location of resource files

### 3. Invalid YAML Syntax

**Error**: \`yaml: line X: could not find expected ':'\`

**Cause**: YAML indentation error

**Fix**: Check indentation (use 2 spaces, not tabs)

### 4. Patch Doesn't Apply

**Symptom**: Patch ignored, no changes

**Cause**: Patch target doesn't match resource

**Fix**: Verify name and kind match exactly

### 5. Wrong Values in Output

**Symptom**: Generated YAML has wrong values

**Fix**:
\`\`\`bash
# Build and inspect
kustomize build overlays/dev | grep -A 5 "replicas"

# Compare with base
kustomize build base
\`\`\`

## Debugging Workflow

### Step 1: Build Locally

\`\`\`bash
kustomize build overlays/dev
\`\`\`

### Step 2: Check Individual Files

\`\`\`bash
# Validate each YAML file
kubectl apply --dry-run=client -f base/deployment.yaml
\`\`\`

### Step 3: Verify Structure

\`\`\`bash
# List files referenced
grep -r "resources:" overlays/dev/
grep -r "bases:" overlays/dev/
\`\`\`

### Step 4: Test Incrementally

\`\`\`bash
# Start with minimal kustomization
# Add features one at a time
# Test after each addition
\`\`\`

## Argo CD Specific Issues

### View Generated Manifests

\`\`\`bash
# See what Argo CD generates
argocd app manifests myapp-dev
\`\`\`

### Force Refresh

\`\`\`bash
# Hard refresh from Git
argocd app get myapp-dev --hard-refresh
\`\`\`

## Best Practices

✅ Test locally before pushing
✅ Use \`kustomize build\` frequently
✅ Keep overlays simple
✅ Document complex patches
✅ Validate YAML syntax

## Key Takeaways

✅ Test builds locally first
✅ Check paths carefully
✅ YAML syntax matters
✅ Debug incrementally
✅ Argo CD shows generated output

## Congratulations!

You've completed Section 6! You now know how to use Kustomize with Argo CD!

Complete **[Lab 4: Using Kustomize with Argo CD](../../labs/lab-04.md)** and **[Lab 5: Multi-Environment with Kustomize](../../labs/lab-05.md)**

Take the [Section 6 Quiz](../../quizzes/section-06-quiz.md)!

Continue to [Section 7: Helm with Argo CD](../section-07/README.md) when ready!)

