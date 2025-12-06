# Lecture 2.1: Argo CD Components (API Server, Repo Server, Controller)

## Introduction

Understanding Argo CD's architecture is crucial for effective troubleshooting, scaling, and advanced usage. In this lecture, we'll explore the three main components that make Argo CD work: the API Server, Repository Server, and Application Controller.

## Argo CD Architecture Overview

Argo CD is a Kubernetes controller that continuously monitors running applications and compares the live state against the desired state in Git. It consists of three main components that work together:

```
┌─────────────────────────────────────────────────────┐
│                    Argo CD                          │
│                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │              │  │              │  │          │ │
│  │  API Server  │  │ Repo Server  │  │Controller│ │
│  │              │  │              │  │          │ │
│  └──────────────┘  └──────────────┘  └──────────┘ │
│         │                  │                │       │
└─────────┼──────────────────┼────────────────┼───────┘
          │                  │                │
          ▼                  ▼                ▼
      Users/CLI          Git Repos      Kubernetes API
```

## Component 1: API Server

### Purpose

The API Server is the primary interface for all Argo CD operations. It's a gRPC/REST server that exposes the Argo CD functionality.

### Key Responsibilities

1. **User Interface**: Serves the web UI
2. **CLI Interface**: Handles CLI commands
3. **Authentication**: Manages user authentication and SSO
4. **RBAC Enforcement**: Enforces role-based access control
5. **Application Management**: CRUD operations for applications
6. **Project Management**: Manages Argo CD projects
7. **Repository Management**: Manages Git repository credentials

### What It Does

- Exposes REST/gRPC API for all Argo CD operations
- Forwards cluster credentials to Repo Server and Controller
- Handles Git webhook events
- Caches data from Repo Server for performance
- Proxies requests to the Kubernetes API

### Example Interactions

```bash
# When you run this command, it talks to the API Server
argocd app list

# When you access the UI, you're connecting to the API Server
https://argocd.example.com

# Creating an app via CLI hits the API Server
argocd app create myapp --repo https://github.com/org/repo
```

## Component 2: Repository Server (Repo Server)

### Purpose

The Repo Server is an internal service that maintains a local cache of Git repositories and generates Kubernetes manifests.

### Key Responsibilities

1. **Git Operations**: Clones and pulls Git repositories
2. **Manifest Generation**: Generates plain Kubernetes manifests from:
   - Plain YAML files
   - Kustomize applications
   - Helm charts
   - Jsonnet
   - Custom tools (plugins)
3. **Caching**: Maintains a local cache of Git repositories
4. **Credential Management**: Uses provided Git credentials

### What It Does

- Clones Git repositories to local cache
- Generates Kubernetes manifests based on repository content
- Runs Helm template or Kustomize build commands
- Caches generated manifests for performance
- Handles private repositories with credentials

### Manifest Generation Flow

```
Git Repo → Repo Server → Generated Manifests → Application Controller
           │
           ├─ For Plain YAML: Read and validate
           ├─ For Kustomize: Run `kustomize build`
           ├─ For Helm: Run `helm template`
           └─ For Jsonnet: Run jsonnet
```

### Example Process

When you deploy a Helm chart:

1. Repo Server clones the Git repository
2. Runs `helm template` with specified values
3. Returns generated YAML to API Server/Controller
4. Controller applies the YAML to Kubernetes

## Component 3: Application Controller

### Purpose

The Application Controller is a Kubernetes controller that continuously monitors running applications and takes action to bring them to the desired state.

### Key Responsibilities

1. **State Monitoring**: Continuously compares desired vs live state
2. **Sync Detection**: Detects when applications are OutOfSync
3. **Auto-Sync**: Automatically syncs applications (if enabled)
4. **Health Assessment**: Monitors application health
5. **Resource Tracking**: Tracks all resources belonging to an application
6. **Event Generation**: Generates Kubernetes events for state changes

### What It Does

- Watches for changes to Application CRDs
- Queries Repo Server for desired state (Git)
- Queries Kubernetes API for live state (cluster)
- Compares states and detects drift
- Executes sync operations when needed
- Runs pre-sync and post-sync hooks
- Updates Application status and sync status

### Reconciliation Loop

```
1. Get Application definition
2. Query Repo Server for desired manifests
3. Query Kubernetes API for live resources
4. Compare desired vs live state
5. If OutOfSync:
   - Update status to OutOfSync
   - If auto-sync enabled, trigger sync
6. If Synced:
   - Update status to Synced
   - Check health status
7. Wait for next reconciliation interval
8. Repeat
```

### Example Behavior

```yaml
# Application Controller continuously checks this
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  source:
    repoURL: https://github.com/org/repo
    path: manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

**What happens**:
1. Controller reads this Application
2. Asks Repo Server: "What should be deployed from this Git repo?"
3. Asks Kubernetes: "What's actually running in the cluster?"
4. Compares the two
5. Takes action based on sync policy

## How Components Work Together

### Example: Deploying an Application

```
User/CLI → API Server → Application Controller → Repo Server → Git
                              │                      │
                              └──────────────────────┘
                                       │
                                       ▼
                              Kubernetes Cluster
```

**Step-by-step**:

1. User creates Application via CLI or UI
2. API Server validates and stores Application CRD
3. Application Controller detects new Application
4. Controller asks Repo Server for manifests from Git
5. Repo Server clones Git repo and generates manifests
6. Controller compares manifests with cluster state
7. Controller applies manifests to cluster (if sync policy allows)
8. Controller continuously monitors for drift

### Example: Detecting Drift

```
Git Changes → Repo Server → Controller → Detects Drift → Updates Status
                                              │
                                              ▼
                                    (If auto-sync) Apply Changes
```

## Component Communication

### API Server ↔ Users

- **Protocol**: REST/gRPC, WebSocket (UI)
- **Authentication**: Local users, SSO, OIDC
- **Authorization**: RBAC policies

### Controller ↔ Kubernetes

- **Protocol**: Kubernetes API
- **Authentication**: Service account token
- **Purpose**: Read/write cluster resources

### Controller ↔ Repo Server

- **Protocol**: gRPC
- **Purpose**: Request manifest generation
- **Data**: Git repo URL, path, credentials

### Repo Server ↔ Git

- **Protocol**: HTTPS, SSH, Git
- **Authentication**: Username/password, SSH keys, tokens
- **Purpose**: Clone and pull repositories

## Deployment Models

### Standard Deployment (Single Cluster)

All components run in the same cluster they manage:

```
Kubernetes Cluster
├── argocd namespace
│   ├── argocd-server (API Server)
│   ├── argocd-repo-server (Repo Server)
│   ├── argocd-application-controller (Controller)
│   └── argocd-redis (Cache)
```

### Multi-Cluster Deployment

Controller and Repo Server run in management cluster, manage multiple target clusters:

```
Management Cluster (Argo CD)
├── argocd-server
├── argocd-repo-server
└── argocd-application-controller
        │
        ├─> Target Cluster 1 (dev)
        ├─> Target Cluster 2 (staging)
        └─> Target Cluster 3 (production)
```

## Scaling Considerations

### API Server

- **Horizontal scaling**: Can run multiple replicas
- **Use case**: High user traffic, many API calls
- **Bottleneck**: Usually not the bottleneck

### Repo Server

- **Horizontal scaling**: Can run multiple replicas
- **Use case**: Many applications, frequent Git changes
- **Bottleneck**: Git operations can be slow

### Application Controller

- **Vertical scaling**: Single active instance (leader election)
- **Sharding**: Can shard applications across multiple controllers
- **Use case**: Hundreds or thousands of applications
- **Bottleneck**: Most likely to become bottleneck

## Key Takeaways

✅ **API Server**: Frontend for users and CLI  
✅ **Repo Server**: Backend for Git and manifest generation  
✅ **Controller**: Brain that does the actual GitOps work  
✅ **Components work together**: Each has a specific role  
✅ **Understanding architecture**: Crucial for troubleshooting

## Common Questions

**Q: Can I run Argo CD without the UI?**  
A: Yes, but you need the API Server for CLI access.

**Q: Can one Repo Server handle multiple Git repos?**  
A: Yes, it caches multiple repositories.

**Q: Does the Controller need direct Git access?**  
A: No, it uses the Repo Server for all Git operations.

**Q: Can I scale all components?**  
A: API Server and Repo Server can scale horizontally. Controller scales vertically or via sharding.

## Next Steps

Now that you understand the components, continue to [Lecture 2.2: Apps, Projects, Repos](./lecture-2.2.md) to learn about the key resources in Argo CD.
