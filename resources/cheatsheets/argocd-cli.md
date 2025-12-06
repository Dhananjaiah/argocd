# Argo CD CLI Cheat Sheet

Quick reference for common Argo CD CLI commands.

## Installation

```bash
# macOS
brew install argocd

# Linux
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd

# Verify
argocd version --client
```

## Login & Authentication

```bash
# Login
argocd login <SERVER> --username admin --password <PASSWORD>
argocd login localhost:8080 --insecure

# Update password
argocd account update-password

# Logout
argocd logout <SERVER>
```

## Application Management

### Create

```bash
# Basic create
argocd app create <APP_NAME> \
  --repo <GIT_URL> \
  --path <PATH> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <NAMESPACE>

# With auto-sync
argocd app create <APP_NAME> \
  --repo <GIT_URL> \
  --path <PATH> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <NAMESPACE> \
  --sync-policy automated \
  --self-heal \
  --auto-prune
```

### List & Get

```bash
# List all apps
argocd app list

# Get app details
argocd app get <APP_NAME>

# Get as YAML/JSON
argocd app get <APP_NAME> -o yaml
argocd app get <APP_NAME> -o json
```

### Sync

```bash
# Sync application
argocd app sync <APP_NAME>

# Sync with prune
argocd app sync <APP_NAME> --prune

# Force sync
argocd app sync <APP_NAME> --force

# Dry run
argocd app sync <APP_NAME> --dry-run
```

### Status & History

```bash
# Wait for healthy
argocd app wait <APP_NAME> --health

# Refresh from Git
argocd app get <APP_NAME> --refresh

# View history
argocd app history <APP_NAME>

# View manifests
argocd app manifests <APP_NAME>

# Diff
argocd app diff <APP_NAME>
```

### Update & Delete

```bash
# Set parameter
argocd app set <APP_NAME> --parameter key=value

# Enable self-heal
argocd app set <APP_NAME> --self-heal

# Delete (keep resources)
argocd app delete <APP_NAME>

# Delete with resources
argocd app delete <APP_NAME> --cascade --yes
```

### Rollback

```bash
# Rollback to previous
argocd app rollback <APP_NAME>

# Rollback to specific revision
argocd app rollback <APP_NAME> <REVISION_ID>
```

## Cluster Management

```bash
# List clusters
argocd cluster list

# Add cluster
argocd cluster add <CONTEXT_NAME>

# Remove cluster
argocd cluster rm <SERVER_URL>
```

## Repository Management

```bash
# List repos
argocd repo list

# Add Git repo
argocd repo add <REPO_URL>

# Add private repo
argocd repo add <REPO_URL> --username <USER> --password <PASSWORD>

# Remove repo
argocd repo rm <REPO_URL>
```

## Common Workflows

### Deploy Application

```bash
argocd app create myapp --repo <URL> --path k8s --dest-server https://kubernetes.default.svc --dest-namespace default
argocd app sync myapp
argocd app wait myapp --health
```

### Update & Sync

```bash
# After Git commit
argocd app get myapp --refresh
argocd app diff myapp
argocd app sync myapp
```

### Troubleshoot

```bash
argocd app get myapp
argocd app diff myapp
argocd app manifests myapp
argocd app get myapp --hard-refresh
```

## Useful Flags

```bash
--server <SERVER>     # Server address
--insecure            # Skip TLS verification
--grpc-web            # Use gRPC web
-o yaml|json          # Output format
```

## Aliases

```bash
alias ac='argocd'
alias acl='argocd app list'
alias acg='argocd app get'
alias acs='argocd app sync'
alias acd='argocd app diff'
```

---

**Tip**: Use `argocd <command> --help` for detailed help on any command!
