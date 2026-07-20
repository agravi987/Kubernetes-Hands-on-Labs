# 📋 Prerequisites

<div align="center">

![Prerequisites](https://img.shields.io/badge/Prerequisites-Setup-blue)
![Time](https://img.shields.io/badge/Time-30%20Minutes-orange)

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║   🛠️  G E T   Y O U R   E N V I R O N M E N T   R E A D Y  ║
║                                                              ║
║      "A craftsman is only as good as their tools."          ║
║                                                    — Rithu  ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Before we dive into Kubernetes, let's make sure your machine is ready. Think of this as the pre-flight checklist before we take off!"* — **Rithu** 🧑‍🏫

---

## 🧠 What You Need

Before starting these labs, make sure you have:

1. **A computer** (Windows, macOS, or Linux) with at least 8GB RAM
2. **Internet connection** (for pulling images and tools)
3. **Admin/root access** on your machine
4. **A terminal** (PowerShell, Terminal, or any shell you like)

---

## 🔧 Tool Installation

### 1. Docker Desktop

Kubernetes needs a container runtime. Docker Desktop is the easiest option.

#### Windows
```powershell
# Download Docker Desktop from:
# https://www.docker.com/products/docker-desktop/

# Or use winget:
winget install Docker.DockerDesktop

# After installation, restart your computer
# Enable "Use the Windows Subsystem for Linux 2" if prompted
```

#### macOS
```bash
# Download Docker Desktop from:
# https://www.docker.com/products/docker-desktop/

# Or use Homebrew:
brew install --cask docker
```

#### Linux (Ubuntu/Debian)
```bash
# Install Docker Engine
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
# Log out and back in for group changes to take effect
```

**Verify Docker:**
```bash
docker --version
# Expected: Docker version 24.x.x or higher

docker run hello-world
# Expected: "Hello from Docker!" message
```

📸 **Screenshot Placeholder:** *[Docker Desktop running successfully]*

---

### 2. kubectl (Kubernetes CLI)

kubectl is the command-line tool for interacting with Kubernetes clusters.

#### Windows
```powershell
# Using winget (recommended)
winget install Kubernetes.kubectl

# Or using Chocolatey
choco install kubernetes-cli

# Or download manually:
# https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
```

#### macOS
```bash
# Using Homebrew
brew install kubectl

# Or using curl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

#### Linux
```bash
# Using package manager
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl

# Or using curl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

**Verify kubectl:**
```bash
kubectl version --client
# Expected: Client Version: version.Info{Major:"1", Minor:"28"...}
```

---

### 3. Minikube

Minikube runs a single-node Kubernetes cluster locally — perfect for learning!

#### Windows
```powershell
# Using winget
winget install Kubernetes.minikube

# Or using Chocolatey
choco install minikube

# Or download from:
# https://minikube.sigs.k8s.io/docs/start/windows/
```

#### macOS
```bash
# Using Homebrew
brew install minikube

# Or using curl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
chmod +x minikube-darwin-amd64
sudo mv minikube-darwin-amd64 /usr/local/bin/minikube
```

#### Linux
```bash
# Using curl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
```

**Verify Minikube:**
```bash
minikube version
# Expected: minikube version: v1.32.x or higher
```

---

### 4. Helm (Optional — Used in Later Labs)

Helm is the package manager for Kubernetes. You'll need it for Labs 20+.

#### Windows
```powershell
# Using Chocolatey
choco install kubernetes-helm

# Or using winget
winget install Helm.Helm
```

#### macOS
```bash
# Using Homebrew
brew install helm
```

#### Linux
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Verify Helm:**
```bash
helm version
# Expected: version.BuildInfo{Version:"v3.x.x"...}
```

---

### 5. eksctl (Optional — Used in Lab 24)

eksctl is the CLI for Amazon EKS. Only needed for Lab 24.

#### Windows
```powershell
# Using Chocolatey
choco install eksctl

# Or using Scoop
scoop install eksctl
```

#### macOS
```bash
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
```

#### Linux
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

**Verify eksctl:**
```bash
eksctl version
# Expected: 0.x.x
```

---

## 🖥️ IDE Setup

### VS Code (Recommended)

Install these extensions for the best experience:

1. **Kubernetes** (`ms-kubernetes-tools.vscode-kubernetes-tools`)
   - Browse clusters, view resources, YAML validation
   
2. **YAML** (`redhat.vscode-yaml`)
   - YAML syntax highlighting and validation

3. **Docker** (`ms-azuretools.vscode-docker`)
   - Dockerfile support and container management

4. **Kubernetes Templates** (`ippon.kubernetes-templates`)
   - Snippets for Kubernetes manifests

📸 **Screenshot Placeholder:** *[VS Code with Kubernetes extensions installed]*

---

## ☁️ Free Kubernetes Cluster Options

If you don't want to use Minikube, here are alternatives:

| Tool | Type | Best For | Resource Usage |
|------|------|----------|----------------|
| **Minikube** | Local cluster | Beginners, all labs | Medium (~2GB RAM) |
| **kind** | Local (Docker-based) | CI/CD, quick tests | Low (~1GB RAM) |
| **K3s** | Lightweight Kubernetes | Edge, IoT, low resources | Very Low (~512MB RAM) |
| **Docker Desktop K8s** | Built-in | Quick start | Medium |
| **EKS (Free Tier)** | Cloud (AWS) | Production-like | Pay-as-you-go |

### Using kind instead of Minikube
```bash
# Install kind
# macOS
brew install kind

# Windows
choco install kind

# Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create a cluster
kind create cluster --name my-cluster

# Verify
kubectl cluster-info --context kind-my-cluster
kubectl get nodes
```

### Using K3s
```bash
# Install K3s (Linux only)
curl -sfL https://get.k3s.io | sh -

# Verify
kubectl get nodes
# Should show a single node in "Ready" state
```

### Using Docker Desktop Kubernetes
```
1. Open Docker Desktop
2. Go to Settings → Kubernetes
3. Check "Enable Kubernetes"
4. Click "Apply & Restart"
5. Wait for installation to complete
6. kubectl is automatically configured
```

---

## ✅ Verification Checklist

Run these commands to make sure everything is ready:

```bash
# Check all tools
echo "=== Docker ===" && docker --version
echo "=== kubectl ===" && kubectl version --client
echo "=== Minikube ===" && minikube version
echo "=== Helm ===" && helm version  # Optional for now
```

Expected output:
```
=== Docker ===
Docker version 24.x.x, build xxxxxxx
=== kubectl ===
Client Version: version.Info{Major:"1", Minor:"28"...}
=== Minikube ===
minikube version: v1.32.x
=== Helm ===
version.BuildInfo{Version:"v3.x.x"...}
```

📸 **Screenshot Placeholder:** *[Terminal showing all tools installed successfully]*

---

## 🚀 Start Your First Cluster

Once everything is installed, test it out:

```bash
# Start Minikube
minikube start --driver=docker

# Check the cluster
kubectl cluster-info

# Check nodes
kubectl get nodes

# You should see:
# NAME       STATUS   ROLES           AGE   VERSION
# minikube   Ready    control-plane   30s   v1.28.x

# Stop Minikube when done
minikube stop
```

> *"If you got this far and your cluster started, you're already ahead of 90% of people who read about Kubernetes but never actually try it. Pat yourself on the back!"* — **Rithu** 🧑‍🏫

---

## 🆘 Troubleshooting

### Common Issues

#### "Docker is not running"
```bash
# Windows/macOS: Start Docker Desktop
# Linux:
sudo systemctl start docker
```

#### "minikube start fails with memory error"
```bash
# Allocate more memory
minikube start --memory=4096 --cpus=2

# Or in Docker Desktop:
# Settings → Resources → Increase Memory to 4GB+
```

#### "kubectl cannot connect to cluster"
```bash
# Make sure Minikube is running
minikube status

# If not, start it
minikube start

# Check kubectl context
kubectl config current-context
# Should show: minikube

# Switch context if needed
kubectl config use-context minikube
```

#### "Permission denied on Linux"
```bash
# Add your user to the docker group
sudo usermod -aG docker $USER
# Then log out and back in
```

---

## 📚 What's Next?

Once your environment is set up, you're ready for:

**[Start Lab 01: Minikube & kubectl Setup →](../01 - Minikube and kubectl Setup/README.md)**

---

*Made with ❤️ by Rithu — "Now let's go break some pods!"* 🧑‍🏫
