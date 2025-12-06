# Lecture 5.3: Sync Options & Best Practices

## Introduction

Argo CD provides many sync options to control behavior. This lecture covers the most important ones and when to use them.

## Common Sync Options

### CreateNamespace

\`\`\`yaml
syncOptions:
  - CreateNamespace=true
\`\`\`

Automatically creates the namespace if it doesn't exist.

### PruneLast

\`\`\`yaml
syncOptions:
  - PruneLast=true
\`\`\`

Deletes resources only after other resources are healthy.

### ApplyOutOfSyncOnly

\`\`\`yaml
syncOptions:
  - ApplyOutOfSyncOnly=true
\`\`\`

Only apply resources that are actually out of sync (performance optimization).

### Validate

\`\`\`yaml
syncOptions:
  - Validate=false
\`\`\`

Skip kubectl validation (use carefully!).

## Retry Logic

\`\`\`yaml
syncPolicy:
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
\`\`\`

Automatically retries failed syncs with exponential backoff.

## Best Practices

✅ Use CreateNamespace for convenience
✅ Use PruneLast for safety
✅ Use retry logic for reliability
✅ Document why specific options used
✅ Test options in dev first

## Key Takeaways

✅ Sync options customize behavior
✅ CreateNamespace is commonly used
✅ Retry logic handles transient failures
✅ Choose options per use case

## Next Steps

Continue to [Lecture 5.4: Handling Drift Safely](./lecture-5.4.md)!)

