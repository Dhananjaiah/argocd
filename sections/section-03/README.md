# Section 3: Lab Setup (Local + Tooling)

## Overview

This section walks you through setting up your local development environment with all the tools needed for the course. By the end, you'll have a complete GitOps-ready workspace.

## Learning Objectives

By the end of this section, you will:
- Install and configure kubectl, Argo CD CLI, Helm, and Kustomize
- Create a local Kubernetes cluster using Kind or Minikube
- Install Argo CD in your cluster
- Access both the Argo CD UI and CLI successfully

## Lectures

1. [3.1 Install Prerequisites (kubectl, argocd CLI, helm, kustomize)](./lecture-3.1.md)
2. [3.2 Create Cluster with Kind/Minikube](./lecture-3.2.md)
3. [3.3 Install Argo CD](./lecture-3.3.md)
4. [3.4 Access UI and CLI Login](./lecture-3.4.md)

## Hands-On Lab

Complete **[Lab 1: Setting Up Your Argo CD Environment](../../labs/lab-01.md)** which covers all topics in this section.

## Key Takeaways

- kubectl is your primary tool for interacting with Kubernetes
- Kind/Minikube provide local Kubernetes clusters perfect for learning
- Argo CD CLI provides command-line access to all Argo CD functions
- Helm and Kustomize are essential tools for managing configurations
- Proper setup is crucial for smooth learning experience

## System Requirements

### Minimum

- 4 CPU cores
- 8GB RAM
- 20GB free disk space
- macOS, Linux, or Windows with WSL2

### Recommended

- 8 CPU cores
- 16GB RAM
- 50GB free disk space
- Stable internet connection

## Tool Versions

This course is compatible with:
- Kubernetes: 1.24+
- Argo CD: 2.8+
- Helm: 3.12+
- Kustomize: 5.0+
- Kind: 0.20+

## Next Steps

After completing this section and Lab 1:
- You'll have a working Argo CD environment
- You'll be ready for your first GitOps deployment
- Continue to [Section 4: First GitOps Deployment](../section-04/README.md)

## Quiz

Complete the [Section 3 Quiz](../../quizzes/section-03-quiz.md) to test your understanding.

## Troubleshooting

Having issues with setup? Check:
- [Lab 1 Troubleshooting Section](../../labs/lab-01.md#troubleshooting)
- [Common Setup Issues](../../resources/troubleshooting/common-issues.md)
- Ensure Docker is running with adequate resources
