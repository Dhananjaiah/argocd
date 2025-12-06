# Labs Overview

Welcome to the hands-on labs! These 15 comprehensive labs will take you from zero to hero with Argo CD and GitOps.

## Lab Structure

Each lab includes:
- **Goal**: What you'll accomplish
- **Prerequisites**: What's needed before starting
- **Step-by-step instructions**: Detailed commands and explanations
- **YAML snippets**: Ready-to-use configurations
- **Verification commands**: How to check your work
- **Troubleshooting**: Common issues and solutions
- **Cleanup**: How to reset for next lab

## Lab Progression

### Beginner Labs (1-5)

**Lab 1**: [Setting Up Your Argo CD Environment](./lab-01.md)  
Set up Kind cluster, install Argo CD, access UI and CLI  
⏱️ 30 minutes | 🎯 Beginner

**Lab 2**: [Your First GitOps Deployment](./lab-02.md)  
Deploy a simple application, manual sync, basic operations  
⏱️ 20 minutes | 🎯 Beginner

**Lab 3**: [Automated Sync and Self-Heal](./lab-03.md)  
Enable auto-sync, prune, self-heal, test drift detection  
⏱️ 25 minutes | 🎯 Beginner

**Lab 4**: [Using Kustomize with Argo CD](./lab-04.md)  
Create base and overlays, deploy with Kustomize  
⏱️ 35 minutes | 🎯 Beginner

**Lab 5**: [Multi-Environment with Kustomize](./lab-05.md)  
Set up dev/staging/prod environments with Kustomize  
⏱️ 40 minutes | 🎯 Intermediate

### Intermediate Labs (6-10)

**Lab 6**: [Deploying Helm Charts](./lab-06.md)  
Use Helm with Argo CD, manage values per environment  
⏱️ 35 minutes | 🎯 Intermediate

**Lab 7**: [App-of-Apps Pattern](./lab-07.md)  
Bootstrap environments, manage multiple applications  
⏱️ 40 minutes | 🎯 Intermediate

**Lab 8**: [RBAC and Multi-Tenancy](./lab-08.md)  
Set up projects, configure RBAC, team isolation  
⏱️ 45 minutes | 🎯 Intermediate

**Lab 9**: [Secrets Management with Sealed Secrets](./lab-09.md)  
Install Sealed Secrets, encrypt secrets, use in apps  
⏱️ 40 minutes | 🎯 Intermediate

**Lab 10**: [Sync Waves and Hooks](./lab-10.md)  
Control deployment order, run pre/post sync jobs  
⏱️ 45 minutes | 🎯 Advanced

### Advanced Labs (11-15)

**Lab 11**: [Notifications with Slack](./lab-11.md)  
Configure Argo CD notifications, Slack integration  
⏱️ 30 minutes | 🎯 Advanced

**Lab 12**: [Image Updater Automation](./lab-12.md)  
Set up Argo CD Image Updater, automate image updates  
⏱️ 35 minutes | 🎯 Advanced

**Lab 13**: [Monitoring and Observability](./lab-13.md)  
Set up metrics, configure Prometheus, create dashboards  
⏱️ 50 minutes | 🎯 Advanced

**Lab 14**: [Multi-Cluster Management](./lab-14.md)  
Add multiple clusters, deploy across clusters  
⏱️ 45 minutes | 🎯 Advanced

**Lab 15**: [Complete Promotion Workflow](./lab-15.md)  
End-to-end workflow: dev → staging → production  
⏱️ 60 minutes | 🎯 Advanced

## Lab Requirements

### System Requirements

- **CPU**: 4 cores minimum
- **RAM**: 8GB minimum (16GB recommended)
- **Disk**: 20GB free space
- **OS**: macOS, Linux, or Windows with WSL2

### Software Requirements

All labs require:
- Docker Desktop (running)
- kubectl
- Kind or Minikube
- Argo CD CLI
- Git

Additional tools for specific labs:
- Helm (Labs 6+)
- Kustomize (Labs 4+)
- kubeseal (Lab 9)

## Getting Started

### Quick Start

```bash
# 1. Start with Lab 1 to set up your environment
cd argocd/labs
open lab-01.md

# 2. Follow instructions step-by-step

# 3. Verify each step before moving to next

# 4. Complete cleanup at end of each lab (unless instructed otherwise)
```

### Lab Tips

1. **Read the entire lab** before starting
2. **Copy commands carefully** - typos cause errors
3. **Wait for resources** to be ready before proceeding
4. **Check verification commands** after each major step
5. **Use troubleshooting section** if issues arise
6. **Don't skip cleanup** to avoid conflicts with next lab

## Lab Dependencies

Some labs build on previous ones:

```
Lab 1 (Setup)
  └── Lab 2 (First Deployment)
       └── Lab 3 (Auto-Sync)
            ├── Lab 4 (Kustomize)
            │    └── Lab 5 (Multi-Env)
            └── Lab 6 (Helm)
                 └── Lab 7 (App-of-Apps)
                      ├── Lab 8 (RBAC)
                      ├── Lab 9 (Secrets)
                      ├── Lab 10 (Waves/Hooks)
                      ├── Lab 11 (Notifications)
                      ├── Lab 12 (Image Updater)
                      ├── Lab 13 (Monitoring)
                      ├── Lab 14 (Multi-Cluster)
                      └── Lab 15 (Promotion)
```

## Time Commitment

- **All Labs**: ~9 hours total
- **Beginner Track** (Labs 1-5): ~2.5 hours
- **Intermediate Track** (Labs 6-10): ~3.5 hours  
- **Advanced Track** (Labs 11-15): ~3 hours

## Lab Completion Checklist

Track your progress:

- [ ] Lab 1: Environment Setup ✅
- [ ] Lab 2: First Deployment ✅
- [ ] Lab 3: Auto-Sync ✅
- [ ] Lab 4: Kustomize Basics ✅
- [ ] Lab 5: Multi-Environment ✅
- [ ] Lab 6: Helm Charts ✅
- [ ] Lab 7: App-of-Apps ✅
- [ ] Lab 8: RBAC ✅
- [ ] Lab 9: Sealed Secrets ✅
- [ ] Lab 10: Waves & Hooks ✅
- [ ] Lab 11: Notifications ✅
- [ ] Lab 12: Image Updater ✅
- [ ] Lab 13: Monitoring ✅
- [ ] Lab 14: Multi-Cluster ✅
- [ ] Lab 15: Complete Workflow ✅

## Common Issues Across Labs

### Issue: Kind Cluster Not Starting

```bash
# Check Docker is running
docker ps

# Delete and recreate cluster
kind delete cluster --name argocd-lab
kind create cluster --name argocd-lab
```

### Issue: Can't Access Argo CD UI

```bash
# Restart port-forward
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
```

### Issue: Pods Stuck in Pending

```bash
# Check node resources
kubectl top nodes

# Describe pod to see issue
kubectl describe pod <pod-name>
```

### Issue: Image Pull Errors

```bash
# Check image name and tag
kubectl get pod <pod-name> -o yaml | grep image:

# Check imagePullPolicy
# May need to pre-pull images on Kind
```

## Getting Help

If you're stuck:

1. **Check the troubleshooting section** in the lab
2. **Review logs**: `kubectl logs <pod-name>`
3. **Check events**: `kubectl get events --sort-by='.lastTimestamp'`
4. **Verify prerequisites** are completed
5. **Restart from cleanup** if needed
6. **Review relevant section** lectures for concepts

## Lab Resources

### Cheat Sheets

- [Argo CD CLI Commands](../resources/cheatsheets/argocd-cli.md)
- [kubectl Commands](../resources/cheatsheets/kubectl.md)
- [Kustomize Commands](../resources/cheatsheets/kustomize.md)
- [Helm Commands](../resources/cheatsheets/helm.md)

### Troubleshooting Guides

- [Common Issues](../resources/troubleshooting/common-issues.md)
- [Debug Guide](../resources/troubleshooting/debug-guide.md)
- [FAQ](../resources/troubleshooting/faq.md)

## Lab Environment

All labs use:
- **Cluster**: Kind (Kubernetes in Docker)
- **Kubernetes**: v1.27+
- **Argo CD**: Latest stable
- **Context**: `kind-argocd-lab`

## Best Practices for Labs

### Before Each Lab

```bash
# Verify cluster is running
kubectl cluster-info

# Check Argo CD is healthy
kubectl get pods -n argocd

# Verify you can access UI
argocd login localhost:8080 --insecure
```

### During Labs

```bash
# Keep multiple terminals open
Terminal 1: Lab instructions
Terminal 2: Running commands
Terminal 3: Watching resources (kubectl get pods -w)
Terminal 4: Viewing logs

# Save commands you run
# Take notes on what works/doesn't work
# Screenshot interesting UI views
```

### After Each Lab

```bash
# Complete the cleanup section
# Verify resources are deleted
kubectl get all -A

# Update your progress checklist
# Review what you learned
```

## Next Steps

1. **Start with Lab 1**: [Setting Up Your Argo CD Environment](./lab-01.md)
2. **Complete in order** for best learning experience
3. **Take your time** - understanding is more important than speed
4. **Experiment** - try variations of commands
5. **Have fun!** GitOps is powerful and elegant

## Additional Learning

After completing the labs:
- Try the [Capstone Project](../capstone/README.md)
- Explore [Example Applications](../examples/README.md)
- Review [Advanced Patterns](../resources/advanced-patterns.md)

---

**Ready to get hands-on?** Start with [Lab 1](./lab-01.md) now!
