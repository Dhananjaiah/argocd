# Lab 2: Your First GitOps Deployment

**Difficulty**: Beginner  
**Duration**: 20 minutes  
**Prerequisites**: Lab 1 completed (Argo CD installed)

## Goal

Deploy a simple Nginx application using Argo CD with plain Kubernetes YAML manifests. Understand the basic GitOps workflow.

## What You'll Learn

- How to create an Argo CD Application
- How to perform manual sync
- How to view application status and health
- How to make changes via Git
- How to rollback changes

## Step 1: Create Application Repository

For this lab, we'll use a pre-made example repository. In a real scenario, you'd create your own.

### Option A: Use Example Repository (Recommended for Lab)

We'll use this repository:
```
https://github.com/argoproj/argocd-example-apps
```

This contains sample applications. We'll deploy the `guestbook` app.

### Option B: Create Your Own (Optional)

If you want to create your own:

```bash
# Create directory
mkdir -p ~/argocd-apps/simple-nginx
cd ~/argocd-apps/simple-nginx

# Create deployment YAML
cat <<'EOF' > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
EOF

# Create service YAML
cat <<'EOF' > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
EOF

# Initialize Git
git init
git add .
git commit -m "Initial commit"

# Push to your GitHub (create empty repo first)
git remote add origin https://github.com/YOUR_USERNAME/simple-nginx.git
git push -u origin main
```

For this lab, we'll use Option A for simplicity.

## Step 2: Create Argo CD Application via CLI

```bash
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

**Command breakdown**:
- `guestbook`: Application name
- `--repo`: Git repository URL
- `--path`: Path within the repository
- `--dest-server`: Destination Kubernetes cluster
- `--dest-namespace`: Target namespace

**Expected Output**:
```
application 'guestbook' created
```

## Step 3: View Application Status

### Via CLI

```bash
argocd app get guestbook
```

**Expected Output**:
```
Name:               guestbook
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:8080/applications/guestbook
Repo:               https://github.com/argoproj/argocd-example-apps.git
Target:
Path:               guestbook
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from  (53e28ff)
Health Status:      Missing

GROUP  KIND        NAMESPACE  NAME          STATUS     HEALTH   HOOK  MESSAGE
       Service     default    guestbook-ui  OutOfSync  Missing
apps   Deployment  default    guestbook-ui  OutOfSync  Missing
```

**Notice**:
- **Sync Status**: OutOfSync (because we haven't synced yet)
- **Health Status**: Missing (resources don't exist yet)

### Via UI

1. Open browser to https://localhost:8080
2. You should see the `guestbook` application with status "OutOfSync"
3. Click on the application to see details

## Step 4: Sync Application (Manual)

Now let's deploy the application by syncing it.

### Via CLI

```bash
argocd app sync guestbook
```

**Expected Output**:
```
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2024-01-15T10:30:00+00:00            Service     default         guestbook-ui  OutOfSync  Missing
2024-01-15T10:30:00+00:00   apps  Deployment     default         guestbook-ui  OutOfSync  Missing
2024-01-15T10:30:01+00:00            Service     default         guestbook-ui    Synced  Healthy
2024-01-15T10:30:02+00:00   apps  Deployment     default         guestbook-ui    Synced  Progressing
2024-01-15T10:30:03+00:00   apps  Deployment     default         guestbook-ui    Synced  Healthy

Name:               guestbook
...
Sync Status:        Synced to  (53e28ff)
Health Status:      Healthy
```

### Via UI

1. Click on the `guestbook` application
2. Click "SYNC" button
3. Click "SYNCHRONIZE"
4. Watch the resources being created in real-time!

## Step 5: Verify Deployment

### Check Application Status

```bash
argocd app get guestbook
```

Should show:
- **Sync Status**: Synced
- **Health Status**: Healthy

### Check Kubernetes Resources

```bash
# Check pods
kubectl get pods -l app=guestbook-ui

# Check service
kubectl get svc guestbook-ui

# Check deployment
kubectl get deployment guestbook-ui
```

**Expected Output**:
```
NAME                            READY   STATUS    RESTARTS   AGE
guestbook-ui-6c9c5b6d9d-abc12   1/1     Running   0          1m
```

### Access the Application

```bash
# Port-forward to access
kubectl port-forward svc/guestbook-ui 8081:80
```

Open browser to http://localhost:8081

You should see the Guestbook application!

## Step 6: View Application in UI

In Argo CD UI, click on the application to see:

1. **Resource Tree**: Visual representation of all resources
2. **Network Diagram**: Shows relationships (Service → Deployment → Pod)
3. **Events**: Timeline of what happened
4. **Sync Status**: Shows Git commit hash
5. **Live Manifest**: What's actually running
6. **Desired Manifest**: What's in Git

**Explore**: Click on each resource (Deployment, Service, Pod) to see details.

## Step 7: Make a Change via Git

Now let's see GitOps in action by making a change in Git.

### Fork the Repository (for real changes)

For this lab, we'll simulate by changing replica count using app parameters:

```bash
# Update the application to have more replicas
argocd app set guestbook --parameter replicas=3
```

**Note**: In a real scenario, you'd:
1. Fork the repo
2. Clone it locally
3. Edit the YAML files
4. Commit and push
5. Argo CD would detect the change

### Sync the Change

```bash
argocd app sync guestbook
```

### Verify

```bash
kubectl get pods -l app=guestbook-ui
```

You should now see 3 pods!

## Step 8: View Sync History

```bash
argocd app history guestbook
```

**Expected Output**:
```
ID  DATE                           REVISION
0   2024-01-15 10:30:00 +0000 UTC  (53e28ff)
1   2024-01-15 10:35:00 +0000 UTC  (53e28ff)
```

Shows all sync operations.

## Step 9: Rollback

Let's rollback to the previous state with 1 replica.

```bash
# View history
argocd app history guestbook

# Rollback to revision 0
argocd app rollback guestbook 0
```

**Verify**:
```bash
kubectl get pods -l app=guestbook-ui
```

Should be back to 1 replica!

## Step 10: Delete Application

When done, clean up:

```bash
argocd app delete guestbook
```

Type `y` to confirm.

**Verify deletion**:
```bash
kubectl get all -l app=guestbook-ui
```

Should show no resources.

## Verification Commands

### Application Status
```bash
# Get application info
argocd app get guestbook

# List all applications
argocd app list

# Check sync status
argocd app wait guestbook --sync
```

### Resource Verification
```bash
# All resources
kubectl get all -l app=guestbook-ui

# Pods with details
kubectl get pods -l app=guestbook-ui -o wide

# Describe deployment
kubectl describe deployment guestbook-ui
```

### Logs
```bash
# Application logs
kubectl logs -l app=guestbook-ui --tail=50

# Argo CD logs
kubectl logs -n argocd deployment/argocd-server
```

## Troubleshooting

### Issue: Application Stays "OutOfSync"

**Check Git repository access**:
```bash
argocd app get guestbook | grep Repo
```

**Try manual sync**:
```bash
argocd app sync guestbook
```

**Check for errors**:
```bash
argocd app get guestbook
# Look for error messages
```

### Issue: Resources Not Creating

**Check events**:
```bash
kubectl get events --sort-by='.lastTimestamp'
```

**Check pod logs if exists**:
```bash
kubectl get pods -l app=guestbook-ui
kubectl logs <pod-name>
```

**Check Argo CD logs**:
```bash
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
```

### Issue: Can't Access Application UI

**Check service**:
```bash
kubectl get svc guestbook-ui
```

**Check port-forward**:
```bash
# Kill existing
pkill -f "port-forward.*guestbook"

# Start new
kubectl port-forward svc/guestbook-ui 8081:80
```

### Issue: Pods in CrashLoopBackOff

**Check pod description**:
```bash
kubectl describe pod <pod-name>
```

**Check logs**:
```bash
kubectl logs <pod-name>
```

**Common causes**:
- Image pull issues
- Configuration errors
- Resource limits

## Common First-Time Issues

### 1. **Forgot to Sync**

Symptom: Application shows "OutOfSync"
```bash
argocd app sync guestbook
```

### 2. **Wrong Namespace**

Symptom: Can't find resources
```bash
kubectl get pods -n default
# Make sure you're looking in the right namespace
```

### 3. **Port-Forward Died**

Symptom: Can't access UI
```bash
# Restart port-forward
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
```

## What You've Accomplished

✅ Created your first Argo CD Application  
✅ Performed manual sync  
✅ Viewed application status and health  
✅ Made changes (scaling replicas)  
✅ Performed rollback  
✅ Explored the Argo CD UI  
✅ Understood the basic GitOps workflow  

## Key Concepts Learned

1. **Application**: Defines what to deploy and where
2. **Sync**: Process of applying Git manifests to cluster
3. **OutOfSync**: Git differs from cluster
4. **Synced**: Git matches cluster
5. **Health**: Whether resources are running correctly
6. **Manual Sync**: You trigger sync explicitly

## Next Steps

Continue to:
- **[Lab 3: Automated Sync and Self-Heal](./lab-03.md)** - Enable automated GitOps
- **[Section 5: Sync Policies Deep Dive](../sections/section-05/README.md)** - Learn about auto-sync, prune, and self-heal

## Quick Reference

```bash
# Create app
argocd app create <name> \
  --repo <git-url> \
  --path <path> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <namespace>

# Sync app
argocd app sync <name>

# Get app info
argocd app get <name>

# List apps
argocd app list

# Delete app
argocd app delete <name>

# View history
argocd app history <name>

# Rollback
argocd app rollback <name> <revision>
```

---

**Congratulations!** You've completed your first GitOps deployment with Argo CD!
