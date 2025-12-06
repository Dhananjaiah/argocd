# Complete Argo CD Course - GitOps for Kubernetes

A comprehensive, hands-on Udemy-style course covering Argo CD from beginner to advanced levels. This course takes a practical approach to teaching GitOps principles and Argo CD deployment patterns.

## 🎯 Course Overview

This course will teach you how to implement GitOps workflows using Argo CD, the industry-standard declarative GitOps continuous delivery tool for Kubernetes. You'll learn through hands-on labs, starting from the basics and progressing to advanced production patterns.

### What You'll Learn

- **GitOps Fundamentals**: Understand GitOps principles and why they matter
- **Argo CD Architecture**: Deep dive into components and concepts
- **Deployment Strategies**: Plain YAML, Kustomize, and Helm
- **Multi-Environment Patterns**: Dev, Stage, and Production workflows
- **Advanced Patterns**: App-of-Apps, sync waves, hooks, and health checks
- **Security & RBAC**: Multi-tenancy, projects, and role-based access control
- **Secrets Management**: Sealed Secrets, External Secrets Operator, SOPS
- **Operations**: Monitoring, notifications, backup, and disaster recovery
- **Enterprise Patterns**: OpenShift GitOps and production best practices

### What You'll Build

By the end of this course, you'll have built a complete production-ready GitOps workflow:

- Microservice application (frontend + backend)
- Multi-environment deployment (dev/stage/prod)
- Automated sync and self-healing
- App-of-Apps pattern for environment bootstrapping
- Complete promotion workflow using Git PRs

## 📚 Course Structure

This course contains **17 sections** with **60+ lectures**, **15+ hands-on labs**, and **17 quizzes**.

### Sections

1. **Welcome & GitOps Foundations** - Understanding GitOps and Argo CD
2. **Argo CD Architecture & Concepts** - Deep dive into components
3. **Lab Setup** - Setting up your local environment
4. **First GitOps Deployment** - Deploy your first application
5. **Sync Policies Deep Dive** - Manual vs auto sync, prune, self-heal
6. **Kustomize with Argo CD** - Managing configurations with Kustomize
7. **Helm with Argo CD** - Using Helm charts with GitOps
8. **Multi-Environment GitOps Design** - Dev/Stage/Prod patterns
9. **App-of-Apps & Scale Patterns** - Managing multiple applications
10. **Projects, RBAC & Multi-Tenancy** - Security and access control
11. **Secrets in GitOps** - Safe secret management patterns
12. **Advanced Deploy Control** - Sync waves, hooks, health checks
13. **Notifications & Automation** - Slack, Email, and event-driven ops
14. **Image Update Strategies** - Automated image updates
15. **Observability & Operations** - Monitoring, logging, backup
16. **OpenShift GitOps** - Enterprise GitOps patterns
17. **Capstone Project** - Build a complete end-to-end solution

## 🛠️ Prerequisites

- Basic Kubernetes knowledge (pods, deployments, services)
- Basic Git knowledge
- Command line familiarity
- Docker basics

### Tools Required

- `kubectl` - Kubernetes CLI
- `argocd` - Argo CD CLI
- `helm` - Helm package manager
- `kustomize` - Kubernetes configuration management
- `kind` or `minikube` - Local Kubernetes cluster
- `git` - Version control
- Text editor (VS Code recommended)

## 🚀 Getting Started

1. **Clone this repository**:
   ```bash
   git clone https://github.com/Dhananjaiah/argocd.git
   cd argocd
   ```

2. **Follow Section 3** to set up your local environment

3. **Start with the labs** in sequential order

## 📖 Course Navigation

- **`/sections`** - All lecture content organized by section
- **`/labs`** - Hands-on lab exercises with step-by-step instructions
- **`/capstone`** - Capstone project materials
- **`/quizzes`** - Section quizzes with answers
- **`/examples`** - Sample applications and YAML files

## 🎓 Certification Preparation

This course prepares you for:
- Argo CD certification concepts
- GitOps practitioner skills
- Kubernetes deployment patterns
- Real-world production scenarios

## 💡 Learning Path

**Beginner Track** (Sections 1-5):
Start here if you're new to GitOps and Argo CD.

**Intermediate Track** (Sections 6-11):
Build on fundamentals with practical patterns.

**Advanced Track** (Sections 12-16):
Production-ready patterns and enterprise features.

**Capstone** (Section 17):
Apply everything you've learned in a complete project.

## 🤝 Contributing

Found an issue or want to improve the course? Contributions are welcome!

## 📝 License

This course material is provided for educational purposes.

## 🌟 Acknowledgments

Built with practical experience from production GitOps implementations.

---

**Ready to master Argo CD and GitOps?** Start with [Section 1: Welcome & GitOps Foundations](./sections/section-01/README.md)
