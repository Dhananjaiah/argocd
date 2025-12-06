# Lecture 3.4: Access UI and CLI Login

## Introduction

Argo CD provides two primary interfaces: a web UI for visual management and a CLI for automation and scripting. In this lecture, you'll learn how to access both and perform basic operations.

## Accessing the Argo CD UI

### Prerequisites

- Argo CD installed and running
- Port forward or other access method configured
- Initial admin password retrieved

### Starting Port Forward

In a terminal (keep it running):

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Opening the UI

1. Open your browser
2. Navigate to: https://localhost:8080
3. Accept the security warning (self-signed certificate)

**Security Warning Steps**:
- Chrome: Click "Advanced" → "Proceed to localhost"
- Firefox: Click "Advanced" → "Accept the Risk"
- Safari: Click "Show Details" → "visit this website"

### First Login

**Username**: `admin`

**Password**: Retrieve with:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```

### UI Overview

After logging in, you'll see:

**Top Navigation**:
- **Applications**: Main view of all applications
- **Settings**: Configuration (repositories, clusters, projects)
- **User Info**: Profile and logout

**Applications View**:
- List or grid view of applications
- Search and filter options
- "NEW APP" button to create applications

**Application Details** (click any app):
- Resource tree visualization
- Sync status and health
- Parameters and manifests
- Logs and events

## Using the Argo CD CLI

### Logging in via CLI

#### Option 1: Username/Password

```bash
# Get the initial password
argocd login localhost:8080 --username admin --password <your-password>

# Or login interactively
argocd login localhost:8080

# Enter credentials when prompted:
# Username: admin
# Password: (paste your password)
```

Expected output:
```
'admin:login' logged in successfully
Context 'localhost:8080' updated
```

#### Option 2: Skip TLS Verification (Local Only)

```bash
argocd login localhost:8080 --insecure --username admin
```

**Note**: `--insecure` skips certificate verification. Only use for local development!

#### Option 3: Using Token

```bash
# Get auth token
ARGOCD_TOKEN=$(kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d)

# Login with token
argocd login localhost:8080 --username admin --password $ARGOCD_TOKEN --insecure
```

### Verifying CLI Login

```bash
# Check server version (verifies connection)
argocd version

# Expected output:
# argocd: v2.8.4+c279299
# BuildDate: 2023-09-13T19:23:18Z
# GitCommit: c279299ff5d43cd9f3f39c8d01a8f5e17a83eb59
# GoVersion: go1.20.7
# argocd-server: v2.8.4+c279299
# BuildDate: 2023-09-13T19:22:30Z
# ...

# Check current context
argocd context

# Expected output:
# CURRENT  NAME             SERVER
# *        localhost:8080   localhost:8080
```

## Changing the Admin Password

### Via UI

1. Click user icon (top right)
2. Select "User Info"
3. Click "Update Password"
4. Enter current and new password
5. Click "Save"

### Via CLI

```bash
# Update admin password
argocd account update-password

# Prompts:
# *** Enter current password:
# *** Enter new password:
# *** Confirm new password:

# Or non-interactively:
argocd account update-password \
  --current-password <old-password> \
  --new-password <new-password>
```

**Important**: After changing password, login again!

```bash
argocd login localhost:8080 --username admin --password <new-password> --insecure
```

## Essential CLI Commands

### Account Management

```bash
# Get account info
argocd account get --account admin

# List all accounts
argocd account list
```

### Application Management

```bash
# List applications
argocd app list

# Get application details
argocd app get <app-name>

# Create application (we'll do this in next section)
argocd app create <app-name> [options]

# Sync application
argocd app sync <app-name>

# Delete application
argocd app delete <app-name>
```

### Cluster Management

```bash
# List clusters
argocd cluster list

# Add cluster (we'll cover this later)
argocd cluster add <context-name>
```

### Repository Management

```bash
# List repositories
argocd repo list

# Add repository
argocd repo add https://github.com/org/repo

# Remove repository
argocd repo rm https://github.com/org/repo
```

### Project Management

```bash
# List projects
argocd proj list

# Get project details
argocd proj get default
```

## CLI Configuration

### Configuration File Location

The CLI stores configuration in:
```
~/.config/argocd/config
```

View it:
```bash
cat ~/.config/argocd/config

# Example content:
# contexts:
# - name: localhost:8080
#   server: localhost:8080
#   user: admin
# current-context: localhost:8080
# servers:
# - grpc-web-root-path: ""
#   server: localhost:8080
# users:
# - auth-token: eyJhbGc...
#   name: admin
```

### Managing Multiple Contexts

```bash
# List contexts
argocd context

# Switch context
argocd context <context-name>

# Add new context (by logging in to different server)
argocd login <another-server>:443
```

## UI Features Overview

### Applications Page

**Main Features**:
- **List/Grid View**: Toggle between views
- **Search**: Find applications by name
- **Filter**: Filter by sync status, health status, project
- **Sort**: Sort by name, sync status, health
- **New App**: Create new application
- **Refresh**: Refresh application status

### Application Details Page

**Tree View** (default):
- Visual representation of resources
- Shows relationships between resources
- Color-coded health status
- Click resources for details

**Network View**:
- Shows networking between services
- Helpful for microservices architectures

**List View**:
- Table view of all resources
- Sortable and filterable

**Tabs**:
- **Summary**: Overview, sync status, health
- **Parameters**: Application configuration
- **Manifest**: Generated YAML
- **Diff**: Compare desired vs live state
- **Events**: Recent events
- **Logs**: Pod logs

### Settings Page

**Repositories**:
- Manage Git repositories
- Add credentials
- Test connections

**Clusters**:
- Manage target clusters
- View cluster info
- Add new clusters

**Projects**:
- Create and manage projects
- Configure RBAC
- Set restrictions

**Accounts**:
- Manage user accounts
- Update passwords
- Configure roles

**Appearance**:
- UI customization options
- Banner messages

## Common Operations

### Checking Application Status

**UI**:
1. Go to Applications page
2. View sync and health status
3. Click application for details

**CLI**:
```bash
argocd app list
argocd app get myapp
```

### Refreshing Applications

**UI**:
- Click "Refresh" button on app card or details page

**CLI**:
```bash
argocd app get myapp --refresh
```

### Viewing Application Diff

**UI**:
1. Open application
2. Click "App Diff" button
3. Review changes

**CLI**:
```bash
argocd app diff myapp
```

### Syncing Applications

**UI**:
1. Open application
2. Click "Sync" button
3. Select resources (or sync all)
4. Click "Synchronize"

**CLI**:
```bash
argocd app sync myapp
```

## CLI Output Formats

### Default Output

```bash
argocd app list
# Human-readable table
```

### JSON Output

```bash
argocd app list -o json
# JSON format for scripting
```

### YAML Output

```bash
argocd app get myapp -o yaml
# YAML format
```

### Wide Output

```bash
argocd app list -o wide
# More details in table
```

## Enabling Shell Completion

### Bash

```bash
# Add to ~/.bashrc
echo 'source <(argocd completion bash)' >> ~/.bashrc
source ~/.bashrc

# Test (type 'argocd app ' and hit TAB)
argocd app <TAB>
```

### Zsh

```bash
# Add to ~/.zshrc
echo 'source <(argocd completion zsh)' >> ~/.zshrc
source ~/.zshrc
```

### Fish

```bash
argocd completion fish | source
```

## Troubleshooting

### Cannot login via CLI

**Error**: `FATA[0000] dial tcp [::1]:8080: connect: connection refused`

**Solution**:
```bash
# Ensure port forward is running
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Then retry login
argocd login localhost:8080 --insecure
```

### Certificate verification failed

**Error**: `x509: certificate signed by unknown authority`

**Solution**:
```bash
# Use --insecure flag (local only!)
argocd login localhost:8080 --insecure
```

### Wrong password

**Error**: `FATA[0000] rpc error: code = Unauthenticated`

**Solution**:
```bash
# Get password again
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo

# Or reset password (requires cluster access)
argocd admin initial-password -n argocd
```

### UI not loading

**Solution**:
```bash
# Check if port forward is running
ps aux | grep port-forward

# Check if argocd-server pod is running
kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server

# Check logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server
```

## Best Practices

✅ **Change default password** immediately after first login  
✅ **Use CLI for automation** (scripts, CI/CD)  
✅ **Use UI for exploration** (visual feedback)  
✅ **Enable shell completion** for productivity  
✅ **Keep CLI updated** to match server version  
✅ **Use contexts** for multiple Argo CD instances

## Key Takeaways

✅ **Two interfaces**: UI for visual, CLI for automation  
✅ **Port forward required**: For local access  
✅ **Change admin password**: Security best practice  
✅ **CLI login persists**: Saved in config file  
✅ **Both interfaces equivalent**: Use what fits your workflow

## Common Questions

**Q: Can I use both UI and CLI simultaneously?**  
A: Yes! They access the same Argo CD instance.

**Q: Do I need to login every time?**  
A: No, CLI stores auth token. UI uses browser session.

**Q: Can multiple users share admin account?**  
A: Technically yes, but create separate accounts for each user instead.

**Q: How long does CLI login last?**  
A: Until token expires (default: no expiration) or you logout.

**Q: Can I disable the UI?**  
A: Yes, use core installation or scale down argocd-server deployment.

## Next Steps

Congratulations! You've completed Section 3. Your Argo CD environment is now fully set up and you can access it via both UI and CLI.

**Complete Lab 1**: [Setting Up Your Argo CD Environment](../../labs/lab-01.md)

Then continue to [Section 4: First GitOps Deployment](../section-04/README.md) to deploy your first application!

Don't forget to complete the [Section 3 Quiz](../../quizzes/section-03-quiz.md)!
