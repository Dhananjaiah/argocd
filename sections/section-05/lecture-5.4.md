# Lecture 5.4: Handling Drift Safely

## Introduction

Drift is inevitable. The key is handling it safely without disrupting services.

## What Causes Drift?

1. **HPA scaling** - Replicas change
2. **Manual changes** - kubectl commands
3. **Operators** - CRD controllers modify resources
4. **Emergency fixes** - Hotfixes in production

## Handling Expected Drift

### Ignore HPA Changes

\`\`\`yaml
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas
\`\`\`

### Ignore Operator Fields

\`\`\`yaml
spec:
  ignoreDifferences:
  - group: apps
    kind: StatefulSet
    jqPathExpressions:
    - .spec.volumeClaimTemplates[]?.metadata.annotations
\`\`\`

## Handling Unexpected Drift

1. **Detect**: Monitor OutOfSync status
2. **Investigate**: Why did drift occur?
3. **Decide**:
   - If correct: Update Git to match
   - If wrong: Sync to revert

## Best Practices

✅ Configure ignoreDifferences for expected drift
✅ Alert on prolonged OutOfSync
✅ Investigate drift cause
✅ Update Git if drift was intentional
✅ Use self-heal to prevent drift

## Key Takeaways

✅ Some drift is expected
✅ ignoreDifferences handles expected drift
✅ Self-heal prevents unexpected drift
✅ Always investigate drift
✅ Git should reflect reality

## Congratulations!

You've completed Section 5! Continue to [Section 6: Kustomize with Argo CD](../section-06/README.md)!

Complete **[Lab 3: Automated Sync and Self-Heal](../../labs/lab-03.md)**

Take the [Section 5 Quiz](../../quizzes/section-05-quiz.md)!)

