# kubectl Installation Guide

<div align="center">

![kubectl](https://img.shields.io/badge/kubectl-v1.28+-326CE5?logo=kubernetes&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐%20Beginner-brightgreen)
![Time](https://img.shields.io/badge/Time-5%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-100%25%20Free-green)

```
╔══════════════════════════════════════════════════════════════╗
║  🔧 KUBECTL INSTALLATION                                     ║
║     "Your Kubernetes Remote Control"                         ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"kubectl is how you talk to Kubernetes. It's like a walkie-talkie for your cluster — you say 'deploy nginx' and Kubernetes listens!"* — **Rithu** 🧑‍🏫

---

## 📋 What is kubectl?

kubectl (pronounced "cube-cuttle" or "kube-C-T-L") is the **command-line tool** for interacting with Kubernetes clusters. It:

- ✅ Creates, inspects, and manages Kubernetes resources
- ✅ Works with Minikube, kind, kubeadm, and cloud clusters
- ✅ Is essential for all Kubernetes work
- ✅ Supports autocomplete and YAML output

---

# 🪟 Windows Installation

## Option 1: Using winget (Recommended)

```powershell
winget install Kubernetes.kubectl
```

## Option 2: Using Chocolatey

```powershell
choco install kubernetes-cli
```

## Option 3: Manual Download

1. Go to: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
2. Download `kubectl.exe`
3. Add to a folder in your PATH (e.g., `C:\kubectl`)
4. Add that folder to your system PATH

## Verify on Windows:

```powershell
kubectl version --client
# Expected: Client Version: version.Info{Major:"1", Minor:"28"...}
```

🎉 **kubectl is installed on Windows!**

---

# 🍎 macOS Installation

## Option 1: Using Homebrew (Recommended)

```bash
brew install kubectl
```

## Option 2: Using curl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

## Verify on macOS:

```bash
kubectl version --client
# Expected: Client Version: version.Info{Major:"1", Minor:"28"...}
```

🎉 **kubectl is installed on macOS!**

---

# 🐧 Linux Installation

## Option 1: Using Package Manager (Ubuntu/Debian)

```bash
# Update package index
sudo apt-get update

# Install apt-transport-https
sudo apt-get install -y apt-transport-https

# Add Kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install kubectl
sudo apt-get update
sudo apt-get install -y kubectl
```

## Option 2: Using curl (Any Linux)

```bash
# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make it executable
chmod +x kubectl

# Move to PATH
sudo mv kubectl /usr/local/bin/
```

## Verify on Linux:

```bash
kubectl version --client
# Expected: Client Version: version.Info{Major:"1", Minor:"28"...}
```

🎉 **kubectl is installed on Linux!**

---

# 🔧 Optional: Shell Autocomplete

## Bash (Linux/macOS):

```bash
# Enable autocomplete
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc

# Reload shell
source ~/.bashrc
```

## PowerShell (Windows):

```powershell
# Enable autocomplete
kubectl completion powershell | Out-String | Invoke-Expression

# Add to profile (optional)
kubectl completion powershell > $(kubectl completion powershell | Out-Null; $PROFILE)
```

## Zsh (macOS):

```bash
# Enable autocomplete
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
echo 'alias k=kubectl' >> ~/.zshrc
echo 'compdef __start_kubectl k' >> ~/.zshrc

# Reload shell
source ~/.zshrc
```

---

# 🔧 Common kubectl Commands

```bash
# Get cluster info
kubectl cluster-info

# List nodes
kubectl get nodes

# List all pods
kubectl get pods --all-namespaces

# Describe a resource
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>

# Execute into a pod
kubectl exec -it <pod-name> -- /bin/bash

# Apply a YAML file
kubectl apply -f manifest.yaml

# Delete a resource
kubectl delete pod <pod-name>

# View resources in YAML format
kubectl get pod <pod-name> -o yaml

# Watch resources in real-time
kubectl get pods -w

# Scale a deployment
kubectl scale deployment <name> --replicas=3

# View all resources in namespace
kubectl get all
```

---

## ✅ Verification Checklist

```bash
# 1. kubectl is installed
kubectl version --client
# Expected: Client Version: version.Info{Major:"1", Minor:"28"...}

# 2. kubectl can connect to a cluster
# (Requires a running cluster - Minikube, kind, or kubeadm)
kubectl cluster-info
# Expected: Kubernetes control plane is running at https://...

# 3. kubectl can list nodes
kubectl get nodes
# Expected: List of cluster nodes
```

---

## 🚀 Next Steps

After kubectl is installed, install a Kubernetes distribution:

```
Choose your path:
├── Minikube (easiest) → ../minikube/README.md
├── kind (fastest) → ../kind/README.md
└── kubeadm (production-like) → ../kubeadm/README.md
```

**[Install Minikube →](../minikube/README.md)** | **[Install kind →](../kind/README.md)** | **[Install kubeadm →](../kubeadm/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 kubectl is ready! You can now talk to Kubernetes!      ║
║     Time to install a cluster! 🔧                           ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

*Made with ❤️ by Rithu — "kubectl is your best friend!"* 🧑‍🏫
