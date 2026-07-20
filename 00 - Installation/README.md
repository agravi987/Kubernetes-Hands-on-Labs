# Kubernetes Installation Hub

<div align="center">

![Installation](https://img.shields.io/badge/Installation-Hub-blue)
![Platforms](https://img.shields.io/badge/Platforms-Windows%20%7C%20macOS%20%7C%20Linux-brightgreen)
![Cost](https://img.shields.io/badge/Cost-100%25%20Free-green)

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║   🛠️  K U B E R N E T E S   I N S T A L L A T I O N       ║
║                                                              ║
║         Choose Your Path to the Cluster!                    ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Everyone's journey starts differently. Some want the easy road (Minikube), some want the fast road (kind), and some want the real road (kubeadm). Pick what fits your vibe!"* — **Rithu** 🧑‍🏫

---

## 🗺️ Choose Your Path

```
┌─────────────────────────────────────────────────────────────────┐
│                    WHICH OPTION IS FOR YOU?                      │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  🟢 MINIKUBE                                           │    │
│  │     Best for: Beginners, learning, single-node        │    │
│  │     Ease: ⭐⭐⭐⭐⭐ (Easiest)                        │    │
│  │     Resource Usage: Medium (~2GB RAM)                  │    │
│  │     Supports: Docker, VirtualBox, Hyper-V, KVM        │    │
│  │                                                          │    │
│  │     👉 Go to: minikube/README.md                       │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  🔵 kind (Kubernetes IN Docker)                        │    │
│  │     Best for: Quick tests, CI/CD, multi-node clusters │    │
│  │     Ease: ⭐⭐⭐⭐ (Very Easy)                        │    │
│  │     Resource Usage: Low (~1GB RAM)                     │    │
│  │     Supports: Docker, Podman                           │    │
│  │                                                          │    │
│  │     👉 Go to: kind/README.md                           │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  🔴 kubeadm (Company Standard)                         │    │
│  │     Best for: Production-like setup, learning k8s     │    │
│  │     Ease: ⭐⭐⭐ (Moderate)                            │    │
│  │     Resource Usage: High (~4GB RAM per node)           │    │
│  │     Supports: Ubuntu, CentOS, RHEL, Debian            │    │
│  │                                                          │    │
│  │     👉 Go to: kubeadm/README.md                        │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  ☁️  Cloud Options (AWS EKS)                            │    │
│  │     Best for: Production deployments                   │    │
│  │     Ease: ⭐⭐⭐ (Moderate)                            │    │
│  │     Resource Usage: Pay-as-you-go                     │    │
│  │                                                          │    │
│  │     👉 Go to: Lab 24 in the main labs                  │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Quick Comparison

| Feature | Minikube | kind | kubeadm | EKS |
|---------|----------|------|---------|-----|
| **Setup Time** | 5 min | 2 min | 15-30 min | 15-20 min |
| **Multi-Node** | Optional | Yes | Yes | Yes |
| **Cost** | Free | Free | Free | Pay-as-you-go |
| **Best For** | Learning | Testing | Production-like | Production |
| **Persistence** | Yes | Optional | Yes | Yes |
| **Add-ons** | Dashboard, Ingress | Basic | Full control | Managed |
| **Difficulty** | Easy | Easy | Moderate | Moderate |

---

## 🔧 Prerequisites (All Options)

Before installing ANY Kubernetes distribution, you need Docker and kubectl:

```
┌─────────────────────────────────────────────────────────────────┐
│  INSTALLATION ORDER:                                            │
│                                                                  │
│  Step 1: Install Docker           → docker/README.md            │
│  Step 2: Install kubectl         → kubectl/README.md           │
│  Step 3: Choose your cluster:                                   │
│          ├── Minikube (easiest)  → minikube/README.md          │
│          ├── kind (fastest)      → kind/README.md              │
│          └── kubeadm (pro)       → kubeadm/README.md           │
└─────────────────────────────────────────────────────────────────┘
```

### 🐳 Docker (Required)
Docker is the container runtime that Kubernetes uses.

**[→ Install Docker (Windows/macOS/Linux)](docker/README.md)**

### 🔧 kubectl (Required)
kubectl is the CLI tool for talking to Kubernetes.

**[→ Install kubectl (Windows/macOS/Linux)](kubectl/README.md)**

---

## 🚀 Quick Start

```bash
# Clone this repository
git clone https://github.com/your-username/Kubernetes-Hands-On-Labs.git
cd Kubernetes-Hands-On-Labs

# Step 1: Install Docker (if not installed)
cd "00 - Installation/docker"
cat README.md  # Follow the instructions

# Step 2: Install kubectl (if not installed)
cd ../kubectl
cat README.md  # Follow the instructions

# Step 3: Install your Kubernetes distribution
cd ../minikube   # For Minikube (recommended for beginners)
cd ../kind       # For kind (fastest)
cd ../kubeadm    # For kubeadm (production-like)

# Follow the README.md in that directory
```

---

## 🎯 Recommendation

> *"For Labs 01-23, use Minikube or kind. They're free, fast, and perfect for learning. Only use kubeadm if you want the full experience, or EKS if you want to go cloud-native!"* — **Rithu** 🧑‍🏫

```
┌──────────────────────────────────────────────────┐
│  RECOMMENDED PATH:                                │
│                                                    │
│  1. Install Minikube or kind (5 min)             │
│  2. Complete Labs 01-23 (free)                   │
│  3. (Optional) Try kubeadm for deeper learning  │
│  4. (Optional) Use EKS for Lab 24                │
└──────────────────────────────────────────────────┘
```

---

*Made with ❤️ by Rithu — "Pick your weapon and let's go!"* 🧑‍🏫
