# GitHub Repository Structure for Course

This document describes the recommended repository structure for organizing the Argo CD course materials and student lab files.

## Repository Organization

```
argocd/
├── README.md                          # Main course overview
├── .gitignore                         # Git ignore file
├── sections/                          # Lecture content by section
│   ├── section-01/                    # Welcome & GitOps Foundations
│   │   ├── README.md                  # Section overview
│   │   ├── lecture-1.1.md             # Course overview
│   │   ├── lecture-1.2.md             # What is GitOps
│   │   ├── lecture-1.3.md             # Why Argo CD
│   │   └── lecture-1.4.md             # GitOps principles
│   ├── section-02/                    # Argo CD Architecture
│   ├── section-03/                    # Lab Setup
│   ├── section-04/                    # First GitOps Deployment
│   ├── section-05/                    # Sync Policies
│   ├── section-06/                    # Kustomize
│   ├── section-07/                    # Helm
│   ├── section-08/                    # Multi-Environment
│   ├── section-09/                    # App-of-Apps
│   ├── section-10/                    # RBAC & Multi-Tenancy
│   ├── section-11/                    # Secrets
│   ├── section-12/                    # Advanced Deploy
│   ├── section-13/                    # Notifications
│   ├── section-14/                    # Image Updates
│   ├── section-15/                    # Observability
│   ├── section-16/                    # OpenShift GitOps
│   └── section-17/                    # Capstone
├── labs/                              # Hands-on lab exercises
│   ├── README.md                      # Labs overview
│   ├── lab-01.md                      # Setup Argo CD
│   ├── lab-02.md                      # First deployment
│   ├── lab-03.md                      # Auto-sync & self-heal
│   ├── lab-04.md                      # Kustomize basics
│   ├── lab-05.md                      # Multi-env with Kustomize
│   ├── lab-06.md                      # Helm charts
│   ├── lab-07.md                      # App-of-Apps pattern
│   ├── lab-08.md                      # RBAC setup
│   ├── lab-09.md                      # Sealed Secrets
│   ├── lab-10.md                      # Sync waves & hooks
│   ├── lab-11.md                      # Notifications setup
│   ├── lab-12.md                      # Image updater
│   ├── lab-13.md                      # Monitoring
│   ├── lab-14.md                      # Multi-cluster
│   └── lab-15.md                      # Full promotion workflow
├── capstone/                          # Capstone project
│   ├── README.md                      # Project overview
│   ├── architecture.md                # Architecture diagrams
│   ├── app-source/                    # Application source code
│   │   ├── frontend/                  # React frontend
│   │   │   ├── Dockerfile
│   │   │   ├── package.json
│   │   │   └── src/
│   │   ├── backend/                   # Node.js API
│   │   │   ├── Dockerfile
│   │   │   ├── package.json
│   │   │   └── src/
│   │   └── database/                  # PostgreSQL setup
│   │       └── init.sql
│   ├── gitops-config/                 # GitOps configuration repo
│   │   ├── apps/                      # Application definitions
│   │   │   ├── frontend-app.yaml
│   │   │   ├── backend-app.yaml
│   │   │   └── database-app.yaml
│   │   ├── environments/              # Environment-specific configs
│   │   │   ├── dev/
│   │   │   │   ├── kustomization.yaml
│   │   │   │   └── patches/
│   │   │   ├── staging/
│   │   │   │   ├── kustomization.yaml
│   │   │   │   └── patches/
│   │   │   └── production/
│   │   │       ├── kustomization.yaml
│   │   │       └── patches/
│   │   ├── base/                      # Base manifests
│   │   │   ├── frontend/
│   │   │   ├── backend/
│   │   │   └── database/
│   │   └── app-of-apps/               # App-of-Apps configs
│   │       ├── dev-apps.yaml
│   │       ├── staging-apps.yaml
│   │       └── production-apps.yaml
│   ├── helm-charts/                   # Helm chart versions
│   │   ├── microservice-chart/
│   │   │   ├── Chart.yaml
│   │   │   ├── values.yaml
│   │   │   ├── values-dev.yaml
│   │   │   ├── values-staging.yaml
│   │   │   ├── values-production.yaml
│   │   │   └── templates/
│   │   └── README.md
│   └── docs/                          # Capstone documentation
│       ├── phase-1-plain-yaml.md
│       ├── phase-2-kustomize.md
│       ├── phase-3-helm.md
│       └── phase-4-app-of-apps.md
├── examples/                          # Sample applications
│   ├── simple-nginx/                  # Basic YAML deployment
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── kustomize-demo/                # Kustomize example
│   │   ├── base/
│   │   └── overlays/
│   ├── helm-demo/                     # Helm example
│   │   └── my-chart/
│   ├── sync-waves-demo/               # Sync waves example
│   │   └── manifests/
│   ├── hooks-demo/                    # Hooks example
│   │   └── manifests/
│   └── app-of-apps-demo/              # App-of-Apps example
│       └── apps/
├── quizzes/                           # Section quizzes
│   ├── section-01-quiz.md
│   ├── section-02-quiz.md
│   ├── section-03-quiz.md
│   ├── section-04-quiz.md
│   ├── section-05-quiz.md
│   ├── section-06-quiz.md
│   ├── section-07-quiz.md
│   ├── section-08-quiz.md
│   ├── section-09-quiz.md
│   ├── section-10-quiz.md
│   ├── section-11-quiz.md
│   ├── section-12-quiz.md
│   ├── section-13-quiz.md
│   ├── section-14-quiz.md
│   ├── section-15-quiz.md
│   ├── section-16-quiz.md
│   └── section-17-quiz.md
├── resources/                         # Additional resources
│   ├── cheatsheets/
│   │   ├── argocd-cli.md
│   │   ├── kubectl.md
│   │   ├── kustomize.md
│   │   └── helm.md
│   ├── troubleshooting/
│   │   ├── common-issues.md
│   │   ├── debug-guide.md
│   │   └── faq.md
│   ├── diagrams/                      # Architecture diagrams
│   │   ├── gitops-workflow.png
│   │   ├── argocd-architecture.png
│   │   └── app-of-apps-pattern.png
│   └── links.md                       # External resources
└── scripts/                           # Helper scripts
    ├── setup-kind-cluster.sh
    ├── install-argocd.sh
    ├── cleanup.sh
    └── reset-lab.sh
```

## Separate Repositories Pattern

For a real-world setup, you'd typically have multiple repositories:

### Pattern 1: Separate App Code and Config

```
student-app-source/                    # Application source code repo
├── frontend/
├── backend/
├── .github/
│   └── workflows/
│       └── ci.yaml                    # Build and test

student-gitops-config/                 # GitOps configuration repo
├── apps/
├── environments/
└── README.md
```

**Benefits**:
- Different teams can own different repos
- Separation of concerns
- Different access control

### Pattern 2: Monorepo

```
student-argocd-project/                # Single monorepo
├── src/                               # Application code
├── kubernetes/                        # K8s manifests
├── helm/                              # Helm charts
└── argocd/                            # Argo CD apps
```

**Benefits**:
- Easier for small teams
- Simpler to manage
- Atomic changes across code and config

### Pattern 3: GitOps App-of-Apps

```
gitops-infrastructure/                 # Infrastructure repo
├── bootstrap/
│   └── root-app.yaml                  # Points to all other repos
├── clusters/
│   ├── dev-cluster/
│   ├── staging-cluster/
│   └── prod-cluster/
└── apps/
    └── team-apps/
        ├── team-a-apps.yaml
        └── team-b-apps.yaml

team-a-config/                         # Team A's GitOps repo
├── app1/
└── app2/

team-b-config/                         # Team B's GitOps repo
├── app3/
└── app4/
```

**Benefits**:
- True multi-tenancy
- Teams manage their own repos
- Central bootstrap

## File Naming Conventions

### Lecture Files

```
lecture-<section>.<lecture>.md
Examples:
- lecture-1.1.md
- lecture-1.2.md
- lecture-2.1.md
```

### Lab Files

```
lab-<number>.md
Examples:
- lab-01.md (Setup)
- lab-02.md (First deployment)
- lab-03.md (Auto-sync)
```

### Quiz Files

```
section-<number>-quiz.md
Examples:
- section-01-quiz.md
- section-02-quiz.md
```

## Student Fork Structure

Students should fork the course repo and create their own GitOps repo:

```
student-argocd-course/                 # Forked course repo (read)
└── (follow along with lectures)

student-gitops-lab/                    # Student's GitOps repo (write)
├── apps/
│   ├── my-first-app.yaml
│   └── my-second-app.yaml
├── manifests/
│   ├── dev/
│   ├── staging/
│   └── production/
└── README.md
```

## .gitignore

```gitignore
# IDE
.idea/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Kubernetes
*.kubeconfig
kubeconfig

# Secrets (IMPORTANT!)
secrets/
*.key
*.crt
*.pem
credentials.yaml

# Temporary files
tmp/
temp/
*.tmp

# Logs
*.log
logs/

# Build artifacts
dist/
build/
*.tar.gz

# Node modules
node_modules/
package-lock.json

# Python
__pycache__/
*.pyc
venv/
.env

# Terraform
.terraform/
*.tfstate
*.tfstate.backup
```

## README Templates

### Section README Template

```markdown
# Section X: [Section Title]

## Overview
[Brief description]

## Learning Objectives
- Objective 1
- Objective 2

## Lectures
1. [X.1 Title](./lecture-X.1.md)
2. [X.2 Title](./lecture-X.2.md)

## Key Takeaways
- Key point 1
- Key point 2

## Next Steps
Continue to [Section X+1](../section-XX/README.md)

## Quiz
Complete the [Section X Quiz](../../quizzes/section-X-quiz.md)
```

### Lab README Template

```markdown
# Lab X: [Lab Title]

**Difficulty**: Beginner/Intermediate/Advanced
**Duration**: XX minutes
**Prerequisites**: Lab Y completed

## Goal
[What students will accomplish]

## What You'll Learn
- Learning point 1
- Learning point 2

## Step 1: [Step Title]
[Instructions]

## Verification Commands
[How to verify]

## Troubleshooting
[Common issues and solutions]

## Cleanup
[How to clean up resources]

## What You've Accomplished
✅ Item 1
✅ Item 2

## Next Steps
Continue to [Lab X+1](./lab-XX.md)
```

## Branch Strategy

### Course Repository

```
main                                   # Stable course content
├── feature/section-X                  # New sections
└── fix/typo-section-X                 # Bug fixes
```

### Student GitOps Repository

```
main (production)
├── staging
│   └── develop
│       └── feature/new-app
```

## Documentation Standards

### Code Blocks

Use language-specific syntax highlighting:

```yaml
# For YAML
apiVersion: v1
kind: Service
```

```bash
# For commands
kubectl apply -f deployment.yaml
```

### File Creation

Use heredoc style for file creation in labs:

```bash
cat <<'EOF' > deployment.yaml
apiVersion: apps/v1
kind: Deployment
...
EOF
```

### Expected Output

Always show expected output:

```bash
kubectl get pods

# Expected Output:
NAME                    READY   STATUS    RESTARTS   AGE
myapp-abc123-xyz        1/1     Running   0          1m
```

## Version Control

### Tagging Releases

```bash
git tag -a v1.0.0 -m "Course version 1.0.0"
git push origin v1.0.0
```

### Version Tags

- `v1.0.0` - Initial course release
- `v1.1.0` - New section added
- `v1.0.1` - Bug fixes and typos
- `v2.0.0` - Major restructuring

## Contributing Guidelines

Students can contribute via:

1. Typo fixes
2. Additional examples
3. Improved explanations
4. New troubleshooting tips

Submit via Pull Request with description.

## License

Specify license for course materials (e.g., MIT, Creative Commons).

## Links and References

Keep a centralized `resources/links.md` file with all external references.

---

This structure provides a solid foundation for organizing a comprehensive Argo CD course!
