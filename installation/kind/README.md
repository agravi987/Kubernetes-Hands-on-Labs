# kind (Kubernetes IN Docker) Installation Guide

<div align="center">

![kind](https://img.shields.io/badge/kind-v0.22+-blue?logo=docker)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐%20Easy-blue)
![Time](https://img.shields.io/badge/Time-5%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-100%25%20Free-green)

```
╔══════════════════════════════════════════════════════════════╗
║  🔵 KIND INSTALLATION                                        ║
║     "Kubernetes IN Docker — Lightning Fast!"                 ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"kind is like Minikube's speedy cousin. It spins up Kubernetes clusters as Docker containers — super fast, super lightweight, and perfect for quick tests!"* — **Rithu** 🧑‍🏫

---

## 📋 What is kind?

kind (Kubernetes IN Docker) runs Kubernetes clusters **inside Docker containers**. It's:

- ✅ Extremely fast (cluster in ~30 seconds)
- ✅ Supports multi-node clusters
- ✅ Uses very few resources (~1GB RAM)
- ✅ Perfect for CI/CD pipelines
- ✅ Great for testing Kubernetes features

---

## 🆚 kind vs Minikube

| Feature | kind | Minikube |
|---------|------|----------|
| **Speed** | ⚡ Faster (~30s) | 🐢 Slower (~2min) |
| **Multi-Node** | ✅ Easy | ⚠️ Needs config |
| **Resource Usage** | 🟢 Lower | 🟡 Higher |
| **Persistence** | ⚠️ Optional | ✅ Yes |
| **Add-ons** | ⚠️ Manual | ✅ Built-in |
| **Dashboard** | ❌ No built-in | ✅ Yes |
| **Best For** | CI/CD, Testing | Learning, Development |

> 💡 **Rithu's Tip:** *"Use kind if you want speed and multi-node clusters. Use Minikube if you want ease and built-in features. Both are great!"*

---

## 🔧 System Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| **CPU** | 2 cores | 4+ cores |
| **RAM** | 2GB | 4GB+ |
| **Disk** | 10GB free | 20GB+ free |
| **Docker** | 20.10+ | Latest |
| **OS** | Windows, macOS, Linux | Any |

---

# 🪟 Windows Installation

## Step 1: Install Docker Desktop

```powershell
# Using winget (recommended)
winget install Docker.DockerDesktop

# Or using Chocolatey
choco install docker-desktop
```

**After installation:**
1. Restart your computer
2. Open Docker Desktop
3. Wait for it to start (green icon in system tray)

```powershell
# Verify Docker
docker --version
docker run hello-world
```

---

## Step 2: Install kubectl

```powershell
# Using winget
winget install Kubernetes.kubectl

# Or using Chocolatey
choco install kubernetes-cli
```

```powershell
kubectl version --client
```

---

## Step 3: Install kind

```powershell
# Using winget
winget install Kubernetes.kind

# Or using Chocolatey
choco install kind

# Or manual download
# Go to: https://kind.sigs.k8s.io/docs/user/quick-start/#installation
# Download kind-windows-amd64.exe
# Rename to kind.exe and add to your PATH
```

```powershell
# Verify kind
kind version
# Expected: kind v0.22.x
```

---

## Step 4: Create a Cluster

```powershell
# Create a basic cluster
kind create cluster

# Or with a custom name
kind create cluster --name my-cluster

# With more nodes
kind create cluster --name multi-node --config - <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF
```

**Expected output:**
```
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.28.3) 🖼
 ✓ Preparing nodes 📦 
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
```

📸 **Screenshot Placeholder:** *[kind cluster created successfully on Windows]*

---

## Step 5: Verify Installation

```powershell
# Check cluster info
kubectl cluster-info
# Expected: Kubernetes control plane is running at https://...

# Check nodes
kubectl get nodes
# Expected: NAME STATUS ROLES AGE VERSION
#           kind-control-plane Ready control-plane 30s v1.28.x
```

🎉 **Congratulations! kind is installed on Windows!**

---

# 🍎 macOS Installation

## Step 1: Install Docker Desktop

```bash
# Using Homebrew
brew install --cask docker

# Or manual download
# Go to: https://www.docker.com/products/docker-desktop/
```

```bash
# Verify Docker
docker --version
docker run hello-world
```

---

## Step 2: Install kubectl

```bash
# Using Homebrew
brew install kubectl
```

```bash
kubectl version --client
```

---

## Step 3: Install kind

```bash
# Using Homebrew
brew install kind

# Or using curl
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-darwin-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

```bash
# Verify kind
kind version
```

---

## Step 4: Create a Cluster

```bash
# Create a basic cluster
kind create cluster

# With more resources
kind create cluster --name my-cluster --wait 5m
```

📸 **Screenshot Placeholder:** *[kind cluster created successfully on macOS]*

---

## Step 5: Verify Installation

```bash
kubectl cluster-info
kubectl get nodes
# Expected: kind-control-plane Ready control-plane ... v1.28.x
```

🎉 **Congratulations! kind is installed on macOS!**

---

# 🐧 Linux Installation

## Step 1: Install Docker

```bash
# Update package index
sudo apt-get update

# Install Docker
sudo apt-get install -y docker.io

# Start and enable Docker
sudo systemctl enable docker
sudo systemctl start docker

# Add your user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

```bash
# Verify Docker
docker --version
docker run hello-world
```

---

## Step 2: Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

```bash
kubectl version --client
```

---

## Step 3: Install kind

```bash
# Download kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64

# Make it executable
chmod +x ./kind

# Move to PATH
sudo mv ./kind /usr/local/bin/kind
```

```bash
# Verify kind
kind version
```

---

## Step 4: Create a Cluster

```bash
# Create a basic cluster
kind create cluster

# With custom name
kind create cluster --name dev-cluster
```

📸 **Screenshot Placeholder:** *[kind cluster created successfully on Linux]*

---

## Step 5: Verify Installation

```bash
kubectl cluster-info
kubectl get nodes
# Expected: kind-control-plane Ready control-plane ... v1.28.x
```

🎉 **Congratulations! kind is installed on Linux!**

---

# 🔧 Multi-Node Cluster

One of kind's best features is easy multi-node clusters:

```bash
# Create a multi-node cluster
cat > kind-multi-node.yaml << 'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
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
EOF

kind create cluster --config kind-multi-node.yaml --name multi-node
```

```bash
# Check nodes
kubectl get nodes
# Expected:
# NAME                 STATUS   ROLES           AGE   VERSION
# kind-control-plane   Ready    control-plane   1m    v1.28.x
# kind-worker          Ready    <none>          1m    v1.28.x
# kind-worker2         Ready    <none>          1m    v1.28.x
```

> 💡 **Rithu's Tip:** *"Multi-node clusters are great for testing things like pod scheduling, DaemonSets, and network policies across multiple nodes!"*

---

# 🔧 Common kind Commands

```bash
# Create a cluster
kind create cluster

# Create with custom name
kind create cluster --name my-cluster

# Create with config file
kind create cluster --config config.yaml

# List all clusters
kind get clusters

# Delete a cluster
kind delete cluster

# Delete by name
kind delete cluster --name my-cluster

# Delete all clusters
kind delete clusters --all

# Get cluster kubeconfig
kind get kubeconfig

# Export kubeconfig to file
kind get kubeconfig > ~/.kube/config

# Load a Docker image into kind
kind load docker-image my-app:latest --name my-cluster

# Build a node image (for custom Kubernetes versions)
kind build node-image --image kindest/node:v1.28.3
```

---

# 🔧 kind Configuration Examples

## Example 1: Cluster with Ingress Support

```yaml
# ingress-cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
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
```

```bash
kind create cluster --config ingress-cluster.yaml
```

## Example 2: Cluster with Specific Kubernetes Version

```yaml
# k8s-127.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: kindest/node:v1.27.3
- role: worker
  image: kindest/node:v1.27.3
```

```bash
kind create cluster --config k8s-127.yaml
```

## Example 3: Cluster with Port Mappings

```yaml
# port-mapping.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30000
    hostPort: 30000
    protocol: TCP
  - containerPort: 30001
    hostPort: 30001
    protocol: TCP
```

---

# 🆘 Troubleshooting

### "Docker is not running"
```bash
# Windows/macOS: Start Docker Desktop
# Linux:
sudo systemctl start docker
```

### "kind create cluster fails"
```bash
# Make sure Docker is running
docker ps

# Try with more verbose output
kind create cluster --name my-cluster --retain 2>&1 | tee kind.log

# Check Docker resources
docker info | grep -i "total memory"
```

### "kubectl cannot connect"
```bash
# Check current context
kubectl config current-context

# Set context to kind cluster
kubectl config use-context kind-kind

# Or export kubeconfig
kind get kubeconfig > ~/.kube/config
```

### "kind cluster stuck in creating"
```bash
# Delete and recreate
kind delete cluster --name my-cluster
kind create cluster --name my-cluster
```

### "Permission denied on Linux"
```bash
# Add your user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

---

## ✅ Verification Checklist

```bash
# 1. Docker is running
docker --version
docker run hello-world

# 2. kubectl is installed
kubectl version --client

# 3. kind is installed
kind version

# 4. Cluster is running
kind create cluster --name test
kubectl get nodes
# Expected: kind-control-plane Ready control-plane

# 5. Cleanup
kind delete cluster --name test
```

---

## 🚀 Next Steps

Once kind is installed, you're ready to start the labs!

```bash
# Export kubeconfig for kubectl
kind get kubeconfig > ~/.kube/config

# Now kubectl works with your kind cluster
kubectl get nodes

# Start the labs
cd ../..
cd "01 - Minikube and kubectl Setup"
cat README.md
```

**[Start Lab 01 →](../../01 - Minikube and kubectl Setup/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 kind is ready! Your lightning-fast Kubernetes cluster   ║
║     is spinning up! ⚡                                      ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

*Made with ❤️ by Rithu — "Speed is the name of the game!"* 🧑‍🏫
