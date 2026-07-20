# kubeadm Installation Guide (Company Standard)

<div align="center">

![kubeadm](https://img.shields.io/badge/kubeadm-v1.28+-red?logo=kubernetes)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Moderate-purple)
![Time](https://img.shields.io/badge/Time-30%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-100%25%20Free-green)

```
╔══════════════════════════════════════════════════════════════╗
║  🔴 KUBEADM INSTALLATION                                    ║
║     "The Industry-Standard Way"                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"kubeadm is how the pros do it. It's the same tool used by kops, GKE, and EKS under the hood. If you want to understand how production Kubernetes really works, this is the way!"* — **Rithu** 🧑‍🏫

---

## 📋 What is kubeadm?

kubeadm is the **official Kubernetes installer** that sets up a production-grade cluster. It's:

- ✅ The industry standard for bare-metal Kubernetes
- ✅ Used by major cloud providers under the hood
- ✅ Full control over every component
- ✅ Supports multi-node, HA clusters
- ✅ Teaches you how Kubernetes really works

---

## 🆚 kubeadm vs Minikube vs kind

| Feature | kubeadm | Minikube | kind |
|---------|---------|----------|------|
| **Purpose** | Production setup | Learning/Dev | Testing/CI |
| **Multi-Node** | ✅ Yes | ⚠️ Optional | ✅ Yes |
| **Control Plane** | Full control | Managed | Managed |
| **Networking** | You configure | Built-in | Built-in |
| **Storage** | You configure | Built-in | Built-in |
| **Difficulty** | ⭐⭐⭐ Moderate | ⭐ Easy | ⭐ Easy |
| **Setup Time** | 15-30 min | 5 min | 2 min |
| **Resource Usage** | High | Medium | Low |

> 💡 **Rithu's Tip:** *"Use kubeadm when you want to understand Kubernetes deeply or need a production-like setup. It's harder, but you learn SO much more!"*

---

## 🔧 System Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| **CPU** | 2 cores | 4+ cores |
| **RAM** | 2GB (control plane) | 4GB+ |
| **Disk** | 20GB | 50GB+ |
| **OS** | Ubuntu 20.04+, Debian 11+, CentOS 8+ | Ubuntu 22.04 |
| **Network** | Stable internet | Stable internet |
| **Swap** | Disabled | Disabled |

---

# 🐧 Ubuntu/Debian Installation

## Step 1: Prepare All Nodes

Run these commands on **ALL nodes** (control plane and workers):

```bash
# Update system
sudo apt-get update && sudo apt-get upgrade -y

# Disable swap (Kubernetes requirement!)
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Set required sysctl params
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

> 💡 **Rithu's Tip:** *"Disabling swap is CRITICAL! Kubernetes won't work properly with swap enabled. This is the #1 mistake beginners make!"*

---

## Step 2: Install Container Runtime (containerd)

```bash
# Install containerd
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install containerd
sudo apt-get update
sudo apt-get install -y containerd.io

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Enable SystemdCgroup
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd
```

📸 **Screenshot Placeholder:** *[containerd installed and configured]*

---

## Step 3: Install kubeadm, kubelet, and kubectl

```bash
# Add Kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install kubeadm, kubelet, kubectl
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# Hold packages to prevent automatic updates
sudo apt-mark hold kubelet kubeadm kubectl
```

```bash
# Verify installations
kubeadm version
kubelet --version
kubectl version --client
```

📸 **Screenshot Placeholder:** *[kubeadm, kubelet, kubectl installed]*

---

## Step 4: Initialize Control Plane

Run this **ONLY on the control plane node**:

```bash
# Initialize the cluster
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --upload-certs
```

**Expected output (save this!):**
```
Your Kubernetes control plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

Then you can join any number of worker nodes by running:

  kubeadm join 192.168.1.100:6443 --token abc123.xyz789 \
    --discovery-token-ca-cert-hash sha256:1234...5678
```

📸 **Screenshot Placeholder:** *[kubeadm init successful]*

> ⚠️ **IMPORTANT:** Save the `kubeadm join` command! You'll need it for worker nodes!

---

## Step 5: Configure kubectl

```bash
# Create kubeconfig directory
mkdir -p $HOME/.kube

# Copy admin config
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

# Fix permissions
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verify
kubectl get nodes
# Expected: NAME STATUS ROLES AGE VERSION
#           control-plane NotReady control-plane 1m v1.28.x
```

---

## Step 6: Install Pod Network (Flannel)

The control plane shows `NotReady` because we need a network plugin:

```bash
# Install Flannel CNI
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Wait a moment, then check
kubectl get nodes
# Expected: NAME STATUS ROLES AGE VERSION
#           control-plane Ready control-plane 2m v1.28.x
```

> 💡 **Rithu's Tip:** *"There are many CNI options: Flannel (simple), Calico (features), Weave (easy). Flannel is great for learning!"*

📸 **Screenshot Placeholder:** *[Control plane node in Ready state]*

---

## Step 7: Join Worker Nodes

On **each worker node**, run the `kubeadm join` command from Step 4:

```bash
# Run this on EACH worker node (use YOUR specific command from Step 4)
sudo kubeadm join 192.168.1.100:6443 --token abc123.xyz789 \
  --discovery-token-ca-cert-hash sha256:1234...5678
```

**Expected output on worker:**
```
This node has joined the cluster:
* Certificate signing request was sent to the control-plane
* The CSR was approved by the certificate authority

Set pre-flight image pull policy to IfNotPresent or IfNotPresent
```

---

## Step 8: Verify Cluster

On the **control plane node**:

```bash
# Check all nodes
kubectl get nodes -o wide
```

Expected output:
```
NAME             STATUS   ROLES           AGE   VERSION   INTERNAL-IP     OS-IMAGE
control-plane    Ready    control-plane   5m    v1.28.x   192.168.1.100   Ubuntu 22.04
worker-1         Ready    <none>          1m    v1.28.x   192.168.1.101   Ubuntu 22.04
worker-2         Ready    <none>          1m    v1.28.x   192.168.1.102   Ubuntu 22.04
```

```bash
# Check system pods
kubectl get pods -n kube-system
```

All pods should be Running:
```
NAME                                         READY   STATUS    RESTARTS   AGE
coredns-xxxxxxxxx-xxxxx                      1/1     Running   0          5m
etcd-control-plane                           1/1     Running   0          5m
kube-apiserver-control-plane                 1/1     Running   0          5m
kube-controller-manager-control-plane        1/1     Running   0          5m
kube-flannel-ds-xxxxx                        1/1     Running   0          1m
kube-proxy-xxxxx                             1/1     Running   0          5m
kube-scheduler-control-plane                 1/1     Running   0          5m
```

📸 **Screenshot Placeholder:** *[Full Kubernetes cluster with multiple nodes]*

---

## Step 9: Test the Cluster

```bash
# Deploy a test app
kubectl create deployment nginx --image=nginx:1.25 --replicas=3

# Expose it
kubectl expose deployment nginx --port=80 --type=NodePort

# Check pods
kubectl get pods -o wide
# Expected: Pods distributed across nodes!

# Get the service URL
kubectl get svc nginx
```

🎉 **Congratulations! You have a production-grade Kubernetes cluster!**

---

# 🔧 Common kubeadm Commands

```bash
# Generate new join token (if you lost the original)
kubeadm token create --print-join-command

# Check cluster status
kubectl cluster-info
kubectl get nodes
kubectl get pods -n kube-system

# Drain a node (for maintenance)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Uncordon a node (make it schedulable again)
kubectl uncordon <node-name>

# Cordon a node (mark as unschedulable)
kubectl cordon <node-name>

# View kubeadm config
kubectl -n kube-system get cm kubeadm-config -o yaml

# Upgrade kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm=1.28.x-xx.x
sudo kubeadm upgrade apply

# Reset cluster (DESTRUCTIVE!)
sudo kubeadm reset -f
sudo rm -rf /etc/cni /opt/cni /var/lib/cni /var/lib/kubelet /etc/kubernetes
sudo iptables -F && sudo iptables -t nat -F
```

---

# 🆘 Troubleshooting

### "Node shows NotReady"
```bash
# Check if CNI plugin is installed
kubectl get pods -n kube-system | grep flannel

# If not, install it
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

### "kubectl cannot connect"
```bash
# Make sure kubeconfig is set
export KUBECONFIG=$HOME/.kube/config

# Or check context
kubectl config current-context
kubectl config use-context kubernetes-admin@kubernetes
```

### "pods stuck in Pending"
```bash
# Check node resources
kubectl describe node <node-name>

# Check events
kubectl get events --sort-by='.lastTimestamp'
```

### "kubeadm init fails"
```bash
# Check if containerd is running
sudo systemctl status containerd

# Check if swap is disabled
free -h
# Should show 0 for swap

# Reset and try again
sudo kubeadm reset -f
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### "Worker can't join cluster"
```bash
# Generate new join token on control plane
kubeadm token create --print-join-command

# Run the new join command on worker
```

---

## ✅ Verification Checklist

```bash
# 1. All tools installed
kubeadm version
kubelet --version
kubectl version --client

# 2. All nodes ready
kubectl get nodes
# Expected: All nodes in Ready state

# 3. System pods running
kubectl get pods -n kube-system
# Expected: All pods Running

# 4. Can deploy apps
kubectl create deployment test --image=nginx
kubectl get pods
# Expected: test pod Running

# 5. Cleanup
kubectl delete deployment test
```

---

## 🚀 Next Steps

Once kubeadm is installed, you're ready for the labs!

```bash
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
║  🎉 You just set up a REAL Kubernetes cluster!              ║
║     That's the same foundation used in production! 🏭      ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

*Made with ❤️ by Rithu — "The hard path is the path to mastery!"* 🧑‍🏫
