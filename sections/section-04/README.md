# Section 4: First GitOps Deployment (Plain YAML)

## Overview

Time to deploy your first application using Argo CD! This section covers the complete workflow of creating a GitOps deployment with plain Kubernetes YAML manifests.

## Learning Objectives

By the end of this section, you will:
- Create a Git repository for your application manifests
- Create your first Argo CD Application resource
- Perform manual synchronization
- Debug common first-time deployment issues
- Understand the complete GitOps workflow

## Lectures

1. [4.1 Create a Simple App Repository](./lecture-4.1.md)
2. [4.2 Create Your First Argo CD Application](./lecture-4.2.md)
3. [4.3 Manual Sync](./lecture-4.3.md)
4. [4.4 Debugging Common First-Time Issues](./lecture-4.4.md)

## Hands-On Lab

Complete **[Lab 2: Your First GitOps Deployment](../../labs/lab-02.md)** which covers all topics in this section.

## Key Takeaways

- Git is your source of truth for all deployments
- Argo CD Application CRD defines what, where, and how to deploy
- Manual sync gives you control over when changes are applied
- The GitOps workflow is: commit → push → sync → verify
- Understanding common errors early saves time later

## Application Lifecycle

```
1. Create Git Repo
   ├── Add Kubernetes manifests
   └── Push to GitHub/GitLab

2. Create Argo CD Application
   ├── Define source (Git repo)
   ├── Define destination (cluster + namespace)
   └── Apply Application CRD

3. Sync Application
   ├── Argo CD fetches manifests from Git
   ├── Generates Kubernetes resources
   └── Applies to cluster

4. Verify Deployment
   ├── Check sync status (Synced/OutOfSync)
   ├── Check health status (Healthy/Degraded)
   └── Verify resources are running

5. Make Changes
   ├── Update manifests in Git
   ├── Commit and push
   └── Sync again (manual or automatic)
```

## Common Patterns

### Pattern 1: Single App Repository

```
my-app/
├── README.md
├── deployment.yaml
├── service.yaml
└── configmap.yaml
```

**Pros**: Simple, easy to understand  
**Cons**: Doesn't scale to multiple apps

### Pattern 2: Monorepo with Multiple Apps

```
k8s-apps/
├── app1/
│   ├── deployment.yaml
│   └── service.yaml
├── app2/
│   ├── deployment.yaml
│   └── service.yaml
└── app3/
    ├── deployment.yaml
    └── service.yaml
```

**Pros**: All apps in one place  
**Cons**: Can become large, all teams have access

### Pattern 3: App per Repository

```
app1-repo/        app2-repo/        app3-repo/
├── k8s/          ├── k8s/          ├── k8s/
```

**Pros**: Team ownership, isolated  
**Cons**: More repos to manage

## Troubleshooting Quick Reference

| Issue | Symptom | Solution |
|-------|---------|----------|
| Application stuck OutOfSync | App shows "OutOfSync" | Run `argocd app sync <app>` |
| Resources not creating | Status shows "Missing" | Check events: `kubectl get events` |
| Health check failing | Status shows "Degraded" | Check pod logs: `kubectl logs <pod>` |
| Can't access Git repo | Sync fails with auth error | Check repo URL and credentials |
| Wrong namespace | Resources in wrong place | Check `destination.namespace` |

## Best Practices

1. **Start Simple**: Begin with plain YAML before adding complexity
2. **One App at a Time**: Master basics before multiple apps
3. **Version Control Everything**: All manifests in Git
4. **Test Locally First**: Use `kubectl apply --dry-run=client`
5. **Check Status Often**: Monitor sync and health status
6. **Read Error Messages**: Argo CD provides detailed feedback

## Next Steps

After completing this section and Lab 2:
- You'll have deployed your first GitOps application
- You'll understand the manual sync workflow
- You'll be ready to explore automated sync policies
- Continue to [Section 5: Sync Policies Deep Dive](../section-05/README.md)

## Quiz

Complete the [Section 4 Quiz](../../quizzes/section-04-quiz.md) to test your understanding.

## Additional Resources

- [Example: Simple Nginx](../../examples/simple-nginx/)
- [Argo CD Application CRD Documentation](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#applications)
