# Lecture 3.2: Create Cluster with Kind/Minikube

## Introduction

To learn Argo CD, you need a Kubernetes cluster. In this lecture, we'll create a local Kubernetes cluster using either Kind (Kubernetes in Docker) or Minikube. Both are excellent options for learning and development.

## Choosing Between Kind and Minikube

### Kind (Kubernetes in Docker)

**Pros**:
- ✅ Fast startup (seconds)
- ✅ Lightweight
- ✅ Multiple clusters easy to manage
- ✅ Great for CI/CD pipelines
- ✅ Uses Docker containers as nodes

**Cons**:
- ❌ Limited LoadBalancer support
- ❌ Requires Docker

**Best for**: Quick iterations, multiple clusters, CI/CD

### Minikube

**Pros**:
- ✅ Feature-rich (addons, dashboard, etc.)
- ✅ LoadBalancer support with tunnel
- ✅ Multiple drivers (Docker, VirtualBox, etc.)
- ✅ Easy to use

**Cons**:
- ❌ Slower startup
- ❌ More resource intensive

**Best for**: Feature exploration, full Kubernetes experience

### Recommendation

**Use Kind** if you want a fast, lightweight option.  
**Use Minikube** if you want more features and don't mind slower startup.

This course will show both, but **Kind is recommended** for simplicity.

## Installing Kind

### macOS

```bash
# Using Homebrew
brew install kind

# Verify installation
kind version
# Output: kind v0.20.0 go1.21.0 darwin/amd64
```

### Linux

```bash
# Download binary
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64

# Make executable
chmod +x ./kind

# Move to PATH
sudo mv ./kind /usr/local/bin/kind

# Verify
kind version
```

### Windows (WSL2)

```bash
# Same as Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```

## Creating a Cluster with Kind

### Basic Cluster

```bash
# Create a simple cluster
kind create cluster

# This creates a cluster named 'kind' by default
# Output:
# Creating cluster "kind" ...
# ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
# ✓ Preparing nodes 📦
# ✓ Writing configuration 📜
# ✓ Starting control-plane 🕹️
# ✓ Installing CNI 🔌
# ✓ Installing StorageClass 💾
# Set kubectl context to "kind-kind"
```

### Named Cluster

```bash
# Create cluster with custom name
kind create cluster --name argocd-lab

# kubectl context will be: kind-argocd-lab
```

### Advanced Cluster with Configuration

Create a configuration file for more control:

```yaml
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: argocd-lab
nodes:
  - role: control-plane
    kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
  - role: worker
  - role: worker
```

Create the cluster:

```bash
kind create cluster --config kind-config.yaml

# This creates:
# - 1 control-plane node
# - 2 worker nodes
# - Port forwarding for ingress (80, 443)
```

### Verify Kind Cluster

```bash
# List clusters
kind get clusters

# Check cluster info
kubectl cluster-info --context kind-argocd-lab

# Get nodes
kubectl get nodes

# Expected output:
# NAME                        STATUS   ROLES           AGE   VERSION
# argocd-lab-control-plane    Ready    control-plane   1m    v1.27.3
# argocd-lab-worker           Ready    <none>          1m    v1.27.3
# argocd-lab-worker2          Ready    <none>          1m    v1.27.3
```

## Installing Minikube

### macOS

```bash
# Using Homebrew
brew install minikube

# Verify
minikube version
# Output: minikube version: v1.31.2
```

### Linux

```bash
# Download binary
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Install
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Verify
minikube version
```

### Windows

```bash
# Using Chocolatey
choco install minikube

# Or download from:
# https://minikube.sigs.k8s.io/docs/start/
```

## Creating a Cluster with Minikube

### Basic Cluster

```bash
# Create cluster with default settings
minikube start

# Output:
# 😄  minikube v1.31.2 on Darwin 13.5
# ✨  Automatically selected the docker driver
# 📌  Using Docker Desktop driver with root privileges
# 👍  Starting control plane node minikube in cluster minikube
# 🚜  Pulling base image ...
# 🔥  Creating docker container (CPUs=2, Memory=4000MB) ...
# 🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
# 🔎  Verifying Kubernetes components...
# 🌟  Enabled addons: storage-provisioner, default-storageclass
# 🏄  Done! kubectl is now configured to use "minikube" cluster
```

### Named Profile

```bash
# Create profile with custom name
minikube start -p argocd-lab

# kubectl context will be: argocd-lab
```

### Advanced Cluster with More Resources

```bash
# Create cluster with more resources
minikube start -p argocd-lab \
  --cpus=4 \
  --memory=8192 \
  --disk-size=50g \
  --kubernetes-version=v1.27.3 \
  --driver=docker

# Options explained:
# --cpus: Number of CPUs (default: 2)
# --memory: Memory in MB (default: 4000)
# --disk-size: Disk size (default: 20000mb)
# --kubernetes-version: K8s version
# --driver: docker, virtualbox, hyperkit, etc.
```

### Verify Minikube Cluster

```bash
# Check status
minikube status -p argocd-lab

# Expected output:
# minikube
# type: Control Plane
# host: Running
# kubelet: Running
# apiserver: Running
# kubeconfig: Configured

# Get cluster info
kubectl cluster-info

# Get nodes
kubectl get nodes

# Expected output:
# NAME       STATUS   ROLES           AGE   VERSION
# minikube   Ready    control-plane   2m    v1.27.3
```

## Configuring kubectl Context

### View Available Contexts

```bash
# List all contexts
kubectl config get-contexts

# Example output:
# CURRENT   NAME              CLUSTER           AUTHINFO          NAMESPACE
# *         kind-argocd-lab   kind-argocd-lab   kind-argocd-lab
#           minikube          minikube          minikube
```

### Switch Context

```bash
# Switch to Kind cluster
kubectl config use-context kind-argocd-lab

# Switch to Minikube
kubectl config use-context minikube

# Verify current context
kubectl config current-context
```

### Set Default Namespace (Optional)

```bash
# Set default namespace for current context
kubectl config set-context --current --namespace=argocd

# Verify
kubectl config view --minify | grep namespace:
```

## Verifying Cluster Functionality

### Test Basic Operations

```bash
# Create test namespace
kubectl create namespace test

# Create test pod
kubectl run nginx --image=nginx --namespace=test

# Check pod status
kubectl get pods -n test

# Expected output:
# NAME    READY   STATUS    RESTARTS   AGE
# nginx   1/1     Running   0          10s

# Delete test resources
kubectl delete namespace test
```

### Check Cluster Components

```bash
# Check system pods
kubectl get pods -n kube-system

# Should see:
# - coredns
# - etcd
# - kube-apiserver
# - kube-controller-manager
# - kube-proxy
# - kube-scheduler

# Check cluster info
kubectl cluster-info

# Check node resources
kubectl top nodes
# (may require metrics-server)
```

## Cluster Management Commands

### Kind

```bash
# List clusters
kind get clusters

# Get cluster kubeconfig
kind get kubeconfig --name argocd-lab

# Export kubeconfig
kind export kubeconfig --name argocd-lab

# Delete cluster
kind delete cluster --name argocd-lab

# Load Docker image into cluster (useful for local images)
kind load docker-image myimage:tag --name argocd-lab
```

### Minikube

```bash
# List profiles
minikube profile list

# Get cluster status
minikube status -p argocd-lab

# Get cluster IP
minikube ip -p argocd-lab

# SSH into cluster
minikube ssh -p argocd-lab

# Stop cluster (preserves state)
minikube stop -p argocd-lab

# Start stopped cluster
minikube start -p argocd-lab

# Delete cluster
minikube delete -p argocd-lab

# Open Kubernetes dashboard
minikube dashboard -p argocd-lab
```

## Recommended Cluster for This Course

For this Argo CD course, use this configuration:

### Kind (Recommended)

```yaml
# argocd-cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: argocd
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30080
        hostPort: 8080
        protocol: TCP
      - containerPort: 30443
        hostPort: 8443
        protocol: TCP
```

Create it:

```bash
kind create cluster --config argocd-cluster.yaml

# Verify
kubectl cluster-info --context kind-argocd
kubectl get nodes
```

### Minikube (Alternative)

```bash
minikube start -p argocd \
  --cpus=4 \
  --memory=8192 \
  --kubernetes-version=v1.27.3

# Verify
minikube status -p argocd
kubectl get nodes
```

## Troubleshooting

### Kind: Docker not running

**Error**: `ERROR: failed to create cluster: running kind with rootless provider requires cgroup v2`

**Solution**:
```bash
# Start Docker Desktop
# OR on Linux:
sudo systemctl start docker
```

### Kind: Port already in use

**Error**: `ERROR: failed to create cluster: failed to listen on port 80`

**Solution**:
```bash
# Find what's using the port
sudo lsof -i :80

# Kill the process or use different ports in config
```

### Minikube: Insufficient resources

**Error**: `Requested memory allocation 8192MB is more than your system has`

**Solution**:
```bash
# Start with fewer resources
minikube start -p argocd --cpus=2 --memory=4096
```

### Minikube: Driver issues

**Error**: `Unable to pick a default driver`

**Solution**:
```bash
# Explicitly specify driver
minikube start -p argocd --driver=docker

# Or install specific driver
# macOS: brew install hyperkit
# Linux: Install VirtualBox
```

### kubectl: Context not found

**Error**: `The connection to the server localhost:8080 was refused`

**Solution**:
```bash
# Export kubeconfig
export KUBECONFIG="$(kind get kubeconfig --name argocd)"

# Or for Minikube
minikube update-context -p argocd
```

## Best Practices

✅ **Use consistent naming** for your clusters  
✅ **Allocate adequate resources** (4 CPU, 8GB RAM recommended)  
✅ **Verify cluster before proceeding** to next steps  
✅ **Keep Docker running** while using the cluster  
✅ **Use configuration files** for reproducibility

## Key Takeaways

✅ **Kind**: Fast, lightweight, perfect for learning  
✅ **Minikube**: Feature-rich, great for exploration  
✅ **Both work well** for this course  
✅ **Cluster is ready** for Argo CD installation  
✅ **kubectl configured** to interact with cluster

## Common Questions

**Q: Can I use both Kind and Minikube?**  
A: Yes! They use different contexts, so you can have both running.

**Q: How much disk space do I need?**  
A: At least 20GB free space for images and cluster data.

**Q: Can I use a cloud provider instead?**  
A: Yes, but local clusters are recommended for learning (free, fast iteration).

**Q: Will the cluster persist after reboot?**  
A: Kind: Yes, if Docker is running. Minikube: Yes, unless you delete it.

**Q: How do I access services running in the cluster?**  
A: Kind: Port forwarding or NodePort. Minikube: `minikube service` or tunnel.

## Next Steps

Now that you have a running Kubernetes cluster, continue to [Lecture 3.3: Install Argo CD](./lecture-3.3.md) to install Argo CD.
