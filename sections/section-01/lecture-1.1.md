# Lecture 1.1: Course Overview & What We'll Build

**Duration**: 5 minutes

## Instructor Script

"Welcome to this comprehensive Argo CD course! I'm excited to guide you through the world of GitOps and show you how Argo CD can transform your Kubernetes deployment workflows.

In this course, we're taking a highly practical approach. We won't just talk about theory – we'll build real applications and deploy them using production-ready patterns.

By the end of this course, you'll have built a complete microservices application with frontend and backend components, deployed across three environments – dev, stage, and production – using GitOps principles. You'll master plain YAML deployments, then level up to Kustomize for environment-specific configurations, and finally work with Helm charts. We'll also implement the App-of-Apps pattern, which is how teams manage dozens or hundreds of applications in production.

This isn't just about learning Argo CD – it's about mastering modern Kubernetes deployment practices that you can take directly into your job or projects.

Let's get started!"

## What You'll Build

### 1. **Simple Web Application**
- Frontend: React/Nginx static site
- Backend: Node.js API
- Database: PostgreSQL

### 2. **Three-Environment Pipeline**
- **Dev**: Fast iteration, auto-sync enabled
- **Stage**: Pre-production testing, manual approval
- **Prod**: Stable releases, strict sync policies

### 3. **Progressive Deployment Patterns**
- **Phase 1**: Plain Kubernetes YAML
- **Phase 2**: Kustomize with base + overlays
- **Phase 3**: Helm charts with values per environment

### 4. **App-of-Apps Pattern**
- Bootstrap entire environments with one command
- Manage multiple microservices
- Team-based application grouping

### 5. **Production Features**
- Automated sync and self-healing
- Secret management (Sealed Secrets)
- Health checks and sync waves
- Slack notifications
- Image update automation

## Course Structure

```
17 Sections
├── Foundations (Sections 1-3)
├── Core Concepts (Sections 4-5)
├── Configuration Management (Sections 6-7)
├── Advanced Patterns (Sections 8-12)
├── Operations (Sections 13-15)
├── Enterprise (Section 16)
└── Capstone Project (Section 17)
```

## Tools We'll Use

- **kubectl**: Kubernetes command-line tool
- **argocd**: Argo CD CLI
- **helm**: Package manager for Kubernetes
- **kustomize**: Configuration management tool
- **kind/minikube**: Local Kubernetes cluster
- **git**: Version control

## Key Takeaways

1. **Hands-On First**: Every concept is followed by practical implementation
2. **Progressive Learning**: Start simple, add complexity gradually
3. **Production-Ready**: Learn patterns used in real production environments
4. **Complete Project**: Build a full microservices deployment pipeline
5. **Best Practices**: Avoid common mistakes and anti-patterns

## Demo Steps

No demo for this lecture – it's an overview. Get ready to dive into GitOps concepts next!

## What's Next?

In the next lecture, we'll explore what GitOps really means in a production environment and why it's becoming the standard for Kubernetes deployments.

---

**Time to Learn**: Let's understand GitOps fundamentals in Lecture 1.2!
