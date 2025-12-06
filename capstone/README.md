# Capstone Project: Full-Stack Microservices with GitOps

## Project Overview

Build a complete full-stack microservices application and deploy it across three environments (dev, staging, production) using Argo CD and GitOps principles.

## Application Architecture

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  Internet                                       │
│                                                 │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
         ┌──────────────┐
         │   Ingress    │
         └──────┬───────┘
                │
        ┌───────┴────────┐
        │                │
        ▼                ▼
┌──────────────┐  ┌──────────────┐
│   Frontend   │  │   Backend    │
│   (React)    │──│   (Node.js)  │
│   nginx:80   │  │   :3000      │
└──────────────┘  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │  PostgreSQL  │
                  │   Database   │
                  └──────────────┘
```

## Application Components

### 1. Frontend Service

**Technology**: React + Nginx  
**Features**:
- Single Page Application (SPA)
- Displays tasks from backend API
- Add, edit, delete tasks
- Responsive design

**Endpoints**:
- `/` - Main application
- `/health` - Health check

### 2. Backend Service

**Technology**: Node.js + Express  
**Features**:
- RESTful API
- CRUD operations for tasks
- Database connection pooling
- Structured logging

**Endpoints**:
- `GET /api/tasks` - List all tasks
- `POST /api/tasks` - Create task
- `PUT /api/tasks/:id` - Update task
- `DELETE /api/tasks/:id` - Delete task
- `GET /health` - Health check

### 3. Database

**Technology**: PostgreSQL  
**Features**:
- Persistent storage
- StatefulSet deployment
- PVC for data persistence
- Init scripts for schema

**Schema**:
```sql
CREATE TABLE tasks (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  completed BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW()
);
```

## Project Phases

### Phase 1: Plain Kubernetes YAML

**Goal**: Deploy using basic Kubernetes manifests

**Structure**:
```
kubernetes/
├── frontend/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── backend/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── secret.yaml
└── database/
    ├── statefulset.yaml
    ├── service.yaml
    └── pvc.yaml
```

**Learning**: Basic Kubernetes resources, manual management

### Phase 2: Kustomize with Multi-Environment

**Goal**: Use Kustomize for environment-specific configurations

**Structure**:
```
kustomize/
├── base/
│   ├── frontend/
│   ├── backend/
│   └── database/
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patches/
    ├── staging/
    │   ├── kustomization.yaml
    │   └── patches/
    └── production/
        ├── kustomization.yaml
        └── patches/
```

**Environment Differences**:
- **Dev**: 1 replica, auto-sync, self-heal
- **Staging**: 2 replicas, manual sync, more resources
- **Production**: 3 replicas, manual sync, resource limits

**Learning**: Configuration management, DRY principles

### Phase 3: Helm Charts

**Goal**: Package application as Helm charts

**Structure**:
```
helm/
├── microservice-app/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── values-dev.yaml
│   ├── values-staging.yaml
│   ├── values-production.yaml
│   └── templates/
│       ├── frontend/
│       ├── backend/
│       └── database/
```

**Learning**: Templating, parameterization, chart management

### Phase 4: App-of-Apps Pattern

**Goal**: Bootstrap entire environments with one application

**Structure**:
```
argocd/
├── bootstrap/
│   └── root-app.yaml
├── apps/
│   ├── frontend-app.yaml
│   ├── backend-app.yaml
│   └── database-app.yaml
└── app-of-apps/
    ├── dev-apps.yaml
    ├── staging-apps.yaml
    └── production-apps.yaml
```

**Learning**: Multi-app management, environment bootstrap

## Environment Configuration

### Development

```yaml
environment: dev
replicas:
  frontend: 1
  backend: 1
resources:
  requests:
    cpu: 100m
    memory: 128Mi
sync:
  automated: true
  selfHeal: true
  prune: true
ingress:
  enabled: false  # Use port-forward
monitoring:
  enabled: false
```

### Staging

```yaml
environment: staging
replicas:
  frontend: 2
  backend: 2
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
sync:
  automated: false
  selfHeal: false
  prune: false
ingress:
  enabled: true
  host: staging.myapp.local
monitoring:
  enabled: true
```

### Production

```yaml
environment: production
replicas:
  frontend: 3
  backend: 3
resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 1000m
    memory: 1Gi
sync:
  automated: false
  selfHeal: true
  prune: false
ingress:
  enabled: true
  host: app.myapp.com
monitoring:
  enabled: true
alerts:
  enabled: true
```

## Promotion Workflow

```
┌──────────────────────────────────────────────────┐
│                                                  │
│  1. Developer commits to feature branch          │
│     git commit -m "Add new feature"             │
│     git push origin feature/new-feature          │
│                                                  │
│  2. CI builds and tests                         │
│     - Run tests                                  │
│     - Build Docker image                         │
│     - Tag: myapp:feature-abc123                  │
│     - Push to registry                           │
│                                                  │
│  3. Update dev environment                       │
│     - Update image tag in git                    │
│     - git commit -m "Deploy to dev"             │
│     - Argo CD auto-syncs                         │
│                                                  │
│  4. Test in dev                                  │
│     - Manual testing                             │
│     - Automated tests                            │
│                                                  │
│  5. Promote to staging                          │
│     - Create PR: dev → staging                   │
│     - Code review                                │
│     - Merge PR                                   │
│     - Manual Argo CD sync                        │
│                                                  │
│  6. Test in staging                             │
│     - QA testing                                 │
│     - Performance testing                        │
│     - Security scanning                          │
│                                                  │
│  7. Promote to production                       │
│     - Create PR: staging → production            │
│     - Approval required (2+ reviewers)           │
│     - Merge PR                                   │
│     - Manual Argo CD sync                        │
│     - Monitor rollout                            │
│                                                  │
│  8. Monitor production                          │
│     - Check metrics                              │
│     - Review logs                                │
│     - Validate deployment                        │
│                                                  │
└──────────────────────────────────────────────────┘
```

## Advanced Features

### 1. Secrets Management

```yaml
# Using Sealed Secrets
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: database-credentials
spec:
  encryptedData:
    password: AgB9x2j... (encrypted)
```

### 2. Sync Waves

```yaml
# Database first (wave 0)
apiVersion: v1
kind: Service
metadata:
  name: database
  annotations:
    argocd.argoproj.io/sync-wave: "0"

# Backend second (wave 1)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  annotations:
    argocd.argoproj.io/sync-wave: "1"

# Frontend last (wave 2)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  annotations:
    argocd.argoproj.io/sync-wave: "2"
```

### 3. Hooks

```yaml
# Database migration job
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: migrate/migrate
        command: ["migrate", "-path", "/migrations", "-database", "$DB_URL", "up"]
```

### 4. Health Checks

```yaml
# Custom health check
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: backend
spec:
  source:
    path: backend
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas
  
  # Custom health assessment
  health:
    - group: apps
      kind: Deployment
      check: |
        hs = {}
        if obj.status ~= nil then
          if obj.status.updatedReplicas == obj.spec.replicas then
            hs.status = "Healthy"
            hs.message = "All replicas ready"
            return hs
          end
        end
        hs.status = "Progressing"
        hs.message = "Waiting for replicas"
        return hs
```

### 5. Notifications

```yaml
# Slack notification on sync
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
data:
  trigger.on-sync-succeeded: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-sync-succeeded]
  
  template.app-sync-succeeded: |
    message: |
      Application {{.app.metadata.name}} synced successfully!
      Environment: {{.app.metadata.labels.environment}}
      Revision: {{.app.status.sync.revision}}
```

## Monitoring and Observability

### Metrics

```yaml
# ServiceMonitor for Prometheus
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: backend
spec:
  selector:
    matchLabels:
      app: backend
  endpoints:
  - port: metrics
    path: /metrics
```

### Logging

```yaml
# Fluentd sidecar
containers:
- name: backend
  image: myapp/backend:1.0
- name: fluentd
  image: fluent/fluentd:v1.14
  volumeMounts:
  - name: logs
    mountPath: /var/log
```

### Tracing

```yaml
# OpenTelemetry instrumentation
env:
- name: OTEL_EXPORTER_OTLP_ENDPOINT
  value: "http://jaeger:4317"
- name: OTEL_SERVICE_NAME
  value: "backend"
```

## Success Criteria

At the end of the capstone, students should be able to:

✅ **Deploy** a multi-tier application using Argo CD  
✅ **Manage** three environments (dev, staging, prod)  
✅ **Use** Kustomize for environment-specific configs  
✅ **Create** Helm charts for application packaging  
✅ **Implement** App-of-Apps for environment bootstrap  
✅ **Handle** secrets securely with Sealed Secrets  
✅ **Control** deployment order with sync waves  
✅ **Run** pre/post sync jobs with hooks  
✅ **Promote** changes through environments via PRs  
✅ **Rollback** deployments when issues occur  
✅ **Monitor** application health and sync status  
✅ **Receive** notifications on sync events  

## Project Deliverables

### 1. Source Code Repositories

- Application source code repo (frontend, backend)
- GitOps configuration repo (manifests, charts)

### 2. Documentation

- Architecture diagram
- Deployment guide
- Runbook for operations
- Troubleshooting guide

### 3. GitOps Manifests

- Plain YAML version
- Kustomize version with overlays
- Helm chart version
- App-of-Apps configuration

### 4. CI/CD Pipelines

- Build pipeline (CI)
- GitOps update pipeline
- Promotion workflow

### 5. Demo Video

- Live demonstration of:
  - Making a code change
  - Watching it deploy to dev
  - Promoting to staging
  - Final promotion to production
  - Performing a rollback

## Estimated Timeline

- **Phase 1** (Plain YAML): 2 hours
- **Phase 2** (Kustomize): 3 hours
- **Phase 3** (Helm): 3 hours
- **Phase 4** (App-of-Apps): 2 hours
- **Advanced Features**: 4 hours
- **Documentation**: 2 hours
- **Total**: ~16 hours

## Next Steps

1. Start with [Capstone Phase 1: Plain YAML](./docs/phase-1-plain-yaml.md)
2. Review [Application Source Code](./app-source/README.md)
3. Set up your [GitOps Repository](./gitops-config/README.md)

---

**Ready to build something awesome?** Let's get started with the capstone project!
