# Lecture 3.1: Install Prerequisites (kubectl, argocd CLI, helm, kustomize)

## Introduction

Before we can start working with Argo CD, we need to set up our local development environment with the necessary tools. This lecture covers installing and verifying all prerequisites for the course.

## Prerequisites Overview

We'll install four essential tools:

1. **kubectl** - Kubernetes command-line tool
2. **argocd CLI** - Argo CD command-line tool
3. **helm** - Kubernetes package manager
4. **kustomize** - Kubernetes configuration management tool

Additionally, you'll need:
- **Docker** - For running local Kubernetes clusters
- **Git** - For version control (likely already installed)

## System Requirements

### Minimum Requirements

- **CPU**: 4 cores
- **RAM**: 8GB
- **Disk**: 20GB free space
- **OS**: macOS, Linux, or Windows with WSL2

### Recommended

- **CPU**: 8 cores
- **RAM**: 16GB
- **Disk**: 50GB free space

## Installing kubectl

### macOS

#### Using Homebrew (Recommended)

```bash
# Install kubectl
brew install kubectl

# Verify installation
kubectl version --client

# Expected output:
# Client Version: v1.28.x
```

#### Using curl

```bash
# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"

# Make it executable
chmod +x kubectl

# Move to PATH
sudo mv kubectl /usr/local/bin/

# Verify
kubectl version --client
```

### Linux

```bash
# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make it executable
chmod +x kubectl

# Move to PATH
sudo mv kubectl /usr/local/bin/

# Verify installation
kubectl version --client
```

### Windows (WSL2)

```bash
# Same as Linux instructions above, or use Chocolatey:
choco install kubernetes-cli

# Verify
kubectl version --client
```

### Verification

```bash
# Check version
kubectl version --client --output=yaml

# Should show client version information
```

## Installing Argo CD CLI

### macOS

#### Using Homebrew (Recommended)

```bash
# Install Argo CD CLI
brew install argocd

# Verify installation
argocd version --client

# Expected output:
# argocd: v2.8.x+sha
```

#### Using curl

```bash
# Download latest release
VERSION=$(curl -L -s https://raw.githubusercontent.com/argoproj/argo-cd/stable/VERSION)
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/download/v$VERSION/argocd-darwin-amd64

# Make it executable
chmod +x argocd

# Move to PATH
sudo mv argocd /usr/local/bin/

# Verify
argocd version --client
```

### Linux

```bash
# Download latest release
VERSION=$(curl -L -s https://raw.githubusercontent.com/argoproj/argo-cd/stable/VERSION)
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/download/v$VERSION/argocd-linux-amd64

# Make it executable
chmod +x argocd

# Move to PATH
sudo mv argocd /usr/local/bin/

# Verify
argocd version --client
```

### Windows (WSL2)

```bash
# Same as Linux, or download from GitHub releases page:
# https://github.com/argoproj/argo-cd/releases
```

### Verification

```bash
# Check version
argocd version --client

# Should show version like: argocd: v2.8.4+c279299
```

## Installing Helm

### macOS

#### Using Homebrew (Recommended)

```bash
# Install Helm
brew install helm

# Verify installation
helm version

# Expected output:
# version.BuildInfo{Version:"v3.12.x", ...}
```

#### Using curl

```bash
# Download and install
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod +x get_helm.sh
./get_helm.sh

# Verify
helm version
```

### Linux

```bash
# Download and install
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod +x get_helm.sh
./get_helm.sh

# Verify
helm version
```

### Windows (WSL2)

```bash
# Same as Linux, or use Chocolatey:
choco install kubernetes-helm

# Verify
helm version
```

### Verification

```bash
# Check version
helm version --short

# List installed charts (should be empty initially)
helm list --all-namespaces
```

## Installing Kustomize

### macOS

#### Using Homebrew (Recommended)

```bash
# Install Kustomize
brew install kustomize

# Verify installation
kustomize version

# Expected output:
# v5.x.x
```

#### Using curl

```bash
# Download latest release
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash

# Move to PATH
sudo mv kustomize /usr/local/bin/

# Verify
kustomize version
```

### Linux

```bash
# Download latest release
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash

# Move to PATH
sudo mv kustomize /usr/local/bin/

# Verify
kustomize version
```

### Windows (WSL2)

```bash
# Same as Linux, or use Chocolatey:
choco install kustomize

# Verify
kustomize version
```

### Note about kubectl Built-in Kustomize

kubectl includes Kustomize functionality:

```bash
# Use kubectl with -k flag for Kustomize
kubectl apply -k ./directory

# Check kubectl's kustomize version
kubectl version --client | grep kustomize
```

**Note**: The standalone Kustomize may have newer features than kubectl's built-in version.

### Verification

```bash
# Check version
kustomize version

# Test basic functionality
mkdir test-kustomize
cd test-kustomize
kustomize create
cat kustomization.yaml
cd ..
rm -rf test-kustomize
```

## Installing Docker

Docker is required for running local Kubernetes clusters.

### macOS

```bash
# Download Docker Desktop from:
# https://www.docker.com/products/docker-desktop

# Or use Homebrew
brew install --cask docker

# Start Docker Desktop from Applications

# Verify
docker --version
docker ps
```

### Linux

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to docker group
sudo usermod -aG docker $USER

# Log out and back in for group changes

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verify
docker --version
docker ps
```

### Windows

```bash
# Download Docker Desktop from:
# https://www.docker.com/products/docker-desktop

# Follow installation wizard
# Ensure WSL2 integration is enabled

# Verify in WSL2
docker --version
docker ps
```

## Installing Git

Most systems already have Git, but if not:

### macOS

```bash
# Using Homebrew
brew install git

# Or install Xcode Command Line Tools
xcode-select --install
```

### Linux

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install git

# Fedora
sudo dnf install git

# Arch
sudo pacman -S git
```

### Windows

```bash
# Download from https://git-scm.com/download/win
# Or use Chocolatey
choco install git
```

### Verification

```bash
git --version
# Should show: git version 2.x.x
```

## Verification Script

Create a script to verify all installations:

```bash
#!/bin/bash

echo "Checking prerequisites..."
echo ""

# Check kubectl
if command -v kubectl &> /dev/null; then
    echo "✅ kubectl: $(kubectl version --client --short 2>/dev/null || kubectl version --client)"
else
    echo "❌ kubectl: NOT FOUND"
fi

# Check argocd CLI
if command -v argocd &> /dev/null; then
    echo "✅ argocd CLI: $(argocd version --client --short 2>/dev/null || echo 'installed')"
else
    echo "❌ argocd CLI: NOT FOUND"
fi

# Check helm
if command -v helm &> /dev/null; then
    echo "✅ helm: $(helm version --short)"
else
    echo "❌ helm: NOT FOUND"
fi

# Check kustomize
if command -v kustomize &> /dev/null; then
    echo "✅ kustomize: $(kustomize version --short 2>/dev/null || kustomize version)"
else
    echo "❌ kustomize: NOT FOUND"
fi

# Check docker
if command -v docker &> /dev/null; then
    echo "✅ docker: $(docker --version)"
else
    echo "❌ docker: NOT FOUND"
fi

# Check git
if command -v git &> /dev/null; then
    echo "✅ git: $(git --version)"
else
    echo "❌ git: NOT FOUND"
fi

echo ""
echo "Verification complete!"
```

Save as `check-tools.sh`, make executable, and run:

```bash
chmod +x check-tools.sh
./check-tools.sh
```

## Optional: Shell Completion

Enable auto-completion for better productivity:

### kubectl Completion

```bash
# Bash
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc

# Zsh
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
source ~/.zshrc

# Fish
kubectl completion fish | source
```

### argocd Completion

```bash
# Bash
echo 'source <(argocd completion bash)' >> ~/.bashrc

# Zsh
echo 'source <(argocd completion zsh)' >> ~/.zshrc

# Fish
argocd completion fish | source
```

### helm Completion

```bash
# Bash
echo 'source <(helm completion bash)' >> ~/.bashrc

# Zsh
echo 'source <(helm completion zsh)' >> ~/.zshrc

# Fish
helm completion fish | source
```

## Troubleshooting

### kubectl: command not found

**Solution**: Ensure kubectl is in your PATH:

```bash
# Check PATH
echo $PATH

# Find kubectl
which kubectl

# Add to PATH if needed
export PATH=$PATH:/usr/local/bin
```

### Permission denied when running Docker

**Linux Solution**:
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Log out and back in, or:
newgrp docker

# Verify
docker ps
```

### Tool version too old

**Solution**: Uninstall old version and reinstall:

```bash
# Example for kubectl
brew uninstall kubectl
brew install kubectl

# Or for manual install, remove and re-download
sudo rm /usr/local/bin/kubectl
# Then reinstall using curl method above
```

## Best Practices

✅ **Use package managers** (Homebrew, apt, etc.) for easy updates  
✅ **Keep tools updated** to get latest features and security fixes  
✅ **Enable shell completion** for productivity  
✅ **Verify installations** before proceeding  
✅ **Document versions** used in your projects

## Key Takeaways

✅ **kubectl**: Essential for Kubernetes interaction  
✅ **argocd CLI**: Manage Argo CD from terminal  
✅ **helm**: Package manager for Kubernetes  
✅ **kustomize**: Configuration management tool  
✅ **Docker**: Required for local Kubernetes clusters  
✅ **All tools verified**: Ready for next steps

## Common Questions

**Q: Do I need all these tools?**  
A: Yes, each tool serves a specific purpose in the GitOps workflow.

**Q: Can I use different versions?**  
A: Yes, but ensure compatibility. Kubernetes 1.24+ is recommended.

**Q: Should I use kubectl's built-in Kustomize?**  
A: For this course, standalone Kustomize is recommended for latest features.

**Q: What if I'm on an M1/M2 Mac?**  
A: Use `arm64` architecture downloads instead of `amd64`, or use Homebrew which auto-detects.

**Q: Can I use Podman instead of Docker?**  
A: Yes, Podman works as a Docker alternative on Linux.

## Next Steps

Now that you have all prerequisites installed, continue to [Lecture 3.2: Create Cluster with Kind/Minikube](./lecture-3.2.md) to set up your local Kubernetes cluster.
