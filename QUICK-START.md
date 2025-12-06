# Complete Argo CD Course - Quick Start Guide

This guide helps you get started with the course quickly and understand its structure.

## 🚀 Quick Start (15 minutes)

### Step 1: Prerequisites

Ensure you have:
- Docker Desktop running (4GB+ RAM)
- Git installed
- Terminal/command line access

### Step 2: Start with Lab 1

```bash
# Clone this repository
git clone https://github.com/Dhananjaiah/argocd.git
cd argocd

# Follow Lab 1 instructions
open labs/lab-01.md
```

### Step 3: Complete the Setup

Lab 1 will guide you through:
- Installing kubectl, Kind, Argo CD CLI
- Creating a local Kubernetes cluster
- Installing Argo CD
- Accessing the UI and CLI

**Time**: ~30 minutes

### Step 4: Deploy Your First App

Complete Lab 2 to:
- Create your first Argo CD Application
- Perform manual sync
- Verify deployment

**Time**: ~20 minutes

### Step 5: Enable Automation

Complete Lab 3 to:
- Enable automated sync
- Configure self-heal and prune
- Test drift detection

**Time**: ~25 minutes

**After these 3 labs, you'll have a working GitOps setup!**

## 📚 Course Structure

### For Complete Beginners

**Week 1: Foundations**
- Section 1: Welcome & GitOps Foundations (theory)
- Section 2: Argo CD Architecture (theory)
- Section 3: Lab Setup + Lab 1 (hands-on)
- Section 4: First Deployment + Lab 2 (hands-on)

**Week 2: Core Concepts**
- Section 5: Sync Policies + Lab 3 (hands-on)
- Section 6: Kustomize + Labs 4-5 (hands-on)

**Week 3: Advanced Topics**
- Section 7: Helm + Lab 6
- Section 8: Multi-Environment + Lab 7
- Section 9: App-of-Apps + Lab 8

**Week 4: Production Ready**
- Section 10-16: RBAC, Secrets, Monitoring, etc.
- Labs 9-15
- Capstone Project

### For Experienced Users

Jump directly to topics of interest:

**Already know GitOps?** → Start at Section 3 (Lab Setup)

**Know Argo CD basics?** → Start at Section 6 (Kustomize)

**Need multi-env patterns?** → Sections 6, 8, 9

**Need production patterns?** → Sections 10-16

**Want a complete project?** → Skip to Capstone (Section 17)

## 🎯 Learning Paths

### Path 1: Application Developer

**Focus**: Deploying applications with GitOps

```
Sections: 1, 3, 4, 5, 6
Labs: 1, 2, 3, 4, 5
Time: ~2 weeks
```

**Skills gained**:
- Deploy apps with Argo CD
- Manage multi-environment configs
- Use Kustomize for configuration

### Path 2: Platform Engineer

**Focus**: Building GitOps platforms

```
Sections: 1-10
Labs: 1-8
Time: ~3 weeks
```

**Skills gained**:
- Design GitOps architecture
- Implement App-of-Apps
- Set up RBAC and multi-tenancy
- Configure automation

### Path 3: DevOps/SRE

**Focus**: Production operations

```
Sections: 1-16
Labs: All 15 labs
Time: ~4 weeks
```

**Skills gained**:
- Complete production setup
- Security and secrets management
- Monitoring and observability
- Multi-cluster management

## 📖 How to Use This Course

### Theoretical Sections

Each section contains:
- Overview and learning objectives
- 3-4 detailed lectures (3-6 minutes each)
- Key takeaways
- Quiz (5 questions)

**How to study**:
1. Read section README
2. Read each lecture in order
3. Take notes on key concepts
4. Complete the quiz

### Hands-On Labs

Each lab contains:
- Clear goal and prerequisites
- Step-by-step commands
- YAML snippets (copy-paste ready)
- Verification commands
- Troubleshooting section
- Cleanup instructions

**How to practice**:
1. Read entire lab first
2. Ensure prerequisites are met
3. Follow steps carefully
4. Verify each step
5. Troubleshoot if issues arise
6. Complete cleanup (unless continuing to next lab)

### Example Applications

Located in `examples/` directory:
- `simple-nginx/` - Basic Kubernetes deployment
- `kustomize-demo/` - Kustomize example
- `helm-demo/` - Helm chart example
- `sync-waves-demo/` - Sync waves example
- `hooks-demo/` - Hooks example
- `app-of-apps-demo/` - App-of-Apps example

**Use these to**:
- See working examples
- Test concepts
- Build your own variations

### Capstone Project

Located in `capstone/` directory.

**The capstone project includes**:
- Full-stack microservices app (frontend + backend + database)
- Multi-environment deployment (dev/stage/prod)
- Progressive complexity (YAML → Kustomize → Helm → App-of-Apps)
- All production patterns

**Time to complete**: ~16 hours

## 🛠️ Tools Reference

### Essential Commands

**Argo CD**:
```bash
argocd app list              # List applications
argocd app get <name>        # Get app details
argocd app sync <name>       # Sync application
argocd app diff <name>       # View differences
```

**kubectl**:
```bash
kubectl get pods             # List pods
kubectl logs <pod>           # View logs
kubectl describe <resource>  # Resource details
kubectl get events           # Cluster events
```

**Kustomize**:
```bash
kustomize build <path>       # Build manifests
kubectl apply -k <path>      # Apply with kustomize
```

**Full reference**: See `resources/cheatsheets/`

### Troubleshooting

**Common issues and solutions**: `resources/troubleshooting/common-issues.md`

**Quick fixes**:
- Cluster not starting → Check Docker resources
- UI not accessible → Restart port-forward
- App OutOfSync → Run `argocd app sync`
- Pods not starting → Check `kubectl logs` and `kubectl describe`

## 📊 Progress Tracking

### Checklist

Use this to track your progress:

**Sections**:
- [ ] Section 1: GitOps Foundations ✅
- [ ] Section 2: Architecture ✅
- [ ] Section 3: Lab Setup ✅
- [ ] Section 4: First Deployment ✅
- [ ] Section 5: Sync Policies ✅
- [ ] Section 6: Kustomize ✅
- [ ] Section 7: Helm ✅
- [ ] Section 8: Multi-Environment ✅
- [ ] Section 9: App-of-Apps ✅
- [ ] Section 10: RBAC ✅
- [ ] Section 11: Secrets ✅
- [ ] Section 12: Advanced Deploy ✅
- [ ] Section 13: Notifications ✅
- [ ] Section 14: Image Updates ✅
- [ ] Section 15: Observability ✅
- [ ] Section 16: OpenShift ✅
- [ ] Section 17: Capstone ✅

**Labs**:
- [ ] Lab 1: Environment Setup ✅
- [ ] Lab 2: First Deployment ✅
- [ ] Lab 3: Auto-Sync ✅
- [ ] Lab 4: Kustomize Basics ✅
- [ ] Lab 5: Multi-Environment ✅
- [ ] Lab 6: Helm Charts ✅
- [ ] Lab 7: App-of-Apps ✅
- [ ] Lab 8: RBAC ✅
- [ ] Lab 9: Sealed Secrets ✅
- [ ] Lab 10: Sync Waves & Hooks ✅
- [ ] Lab 11: Notifications ✅
- [ ] Lab 12: Image Updater ✅
- [ ] Lab 13: Monitoring ✅
- [ ] Lab 14: Multi-Cluster ✅
- [ ] Lab 15: Complete Workflow ✅

**Quizzes**:
- [ ] All 17 section quizzes completed

**Capstone**:
- [ ] Phase 1: Plain YAML
- [ ] Phase 2: Kustomize
- [ ] Phase 3: Helm
- [ ] Phase 4: App-of-Apps

## 🎓 Certification Preparation

This course prepares you for:
- Argo CD concepts and operations
- GitOps practitioner skills
- Kubernetes deployment patterns
- Real-world production scenarios

**Not official certification**, but excellent preparation for:
- Job interviews
- Production implementations
- Team training

## 💡 Study Tips

### 1. **Hands-On First**

Don't just read - practice! GitOps is best learned by doing.

### 2. **Build Your Own**

After completing labs, create your own applications.

### 3. **Break Things**

Intentionally cause errors to understand how to fix them.

### 4. **Take Notes**

Document commands that work for your environment.

### 5. **Join Community**

- [Argo CD Slack](https://argoproj.github.io/community/join-slack/)
- [GitHub Discussions](https://github.com/argoproj/argo-cd/discussions)

### 6. **Review Regularly**

Come back to review concepts as you progress.

### 7. **Share Knowledge**

Teach others what you learn - best way to solidify understanding.

## 🤝 Getting Help

### In This Course

1. Check the specific lab's troubleshooting section
2. Review `resources/troubleshooting/common-issues.md`
3. Check section quizzes for concept review

### External Resources

1. [Argo CD Documentation](https://argo-cd.readthedocs.io/)
2. [Argo CD GitHub](https://github.com/argoproj/argo-cd)
3. [Argo CD Slack Community](https://argoproj.github.io/community/join-slack/)

### Common Questions

**Q: Can I use Minikube instead of Kind?**  
A: Yes! The course works with any local Kubernetes cluster.

**Q: How much time do I need?**  
A: Plan for 15-20 hours to complete everything including capstone.

**Q: Do I need a public GitHub account?**  
A: For some labs, yes. You can use any Git hosting service.

**Q: Can I use this in production?**  
A: The patterns taught are production-ready, but adapt to your needs.

**Q: Is this course official?**  
A: No, but it follows Argo CD best practices and official documentation.

## 🌟 What's Next?

After completing this course:

1. **Build Real Projects**: Apply to your actual work
2. **Contribute**: Contribute to Argo CD open source
3. **Share**: Teach others or write blog posts
4. **Advance**: Explore Argo Workflows, Argo Rollouts, Argo Events
5. **Certify**: Consider official Kubernetes certifications (CKA, CKAD)

## 📝 Course Feedback

Found issues? Have suggestions?
- Open an issue
- Submit a pull request
- Share your feedback

---

**Ready to master Argo CD and GitOps?**

👉 **Start with [Lab 1: Setting Up Your Argo CD Environment](labs/lab-01.md)**

**Or explore:**
- [Section 1: Welcome & GitOps Foundations](sections/section-01/README.md)
- [All Labs Overview](labs/README.md)
- [Capstone Project](capstone/README.md)

---

**Good luck and happy learning! 🚀**
