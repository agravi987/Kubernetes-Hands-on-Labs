# Docker Installation Guide

<div align="center">

![Docker](https://img.shields.io/badge/Docker-v24+-2496ED?logo=docker&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐%20Beginner-brightgreen)
![Time](https://img.shields.io/badge/Time-10%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-100%25%20Free-green)

```
╔══════════════════════════════════════════════════════════════╗
║  🐳 DOCKER INSTALLATION                                      ║
║     "The Foundation of Everything"                           ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Docker is the engine that makes containers work. No Docker, no Kubernetes. It's like trying to drive a car without an engine — technically possible, but not very useful!"* — **Rithu** 🧑‍🏫

---

## 📋 What is Docker?

Docker is a platform for building, running, and sharing **containerized applications**. It:

- ✅ Packages apps with all their dependencies
- ✅ Runs apps consistently across any environment
- ✅ Is required for Minikube, kind, and most Kubernetes setups
- ✅ Is free for personal use

---

# 🪟 Windows Installation

## Step 1: Download Docker Desktop

```
Go to: https://www.docker.com/products/docker-desktop/
Click "Download for Windows"
```

## Step 2: Install Docker Desktop

1. Run the downloaded `.exe` installer
2. Follow the installation wizard
3. **IMPORTANT:** Enable "Use WSL 2 instead of Hyper-V" if prompted
4. Click "Ok" to install

📸 **Screenshot Placeholder:** *[Docker Desktop installer on Windows]*

## Step 3: Start Docker Desktop

1. Open Docker Desktop from Start Menu
2. Wait for the whale icon in system tray to turn green
3. This may take 2-3 minutes

## Step 4: Verify Installation

Open PowerShell and run:

```powershell
docker --version
# Expected: Docker version 24.x.x, build xxxxxxx

docker run hello-world
# Expected: "Hello from Docker!" message
```

📸 **Screenshot Placeholder:** *[Docker running successfully on Windows]*

🎉 **Docker is installed on Windows!**

---

# 🍎 macOS Installation

## For Intel Macs:

```
Go to: https://www.docker.com/products/docker-desktop/
Click "Download for Mac - Intel Chip"
```

## For Apple Silicon Macs (M1/M2/M3):

```
Go to: https://www.docker.com/products/docker-desktop/
Click "Download for Mac - Apple Chip"
```

## Installation Steps:

1. Open the downloaded `.dmg` file
2. Drag Docker to Applications folder
3. Open Docker from Applications
4. Grant necessary permissions
5. Wait for the whale icon to turn green

```bash
# Verify installation
docker --version
docker run hello-world
```

🎉 **Docker is installed on macOS!**

---

# 🐧 Linux Installation

## Ubuntu/Debian:

```bash
# Update package index
sudo apt-get update

# Install prerequisites
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

# Install Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start and enable Docker
sudo systemctl enable docker
sudo systemctl start docker

# Add your user to docker group (avoids using sudo)
sudo usermod -aG docker $USER

# Log out and back in, or run:
newgrp docker
```

## CentOS/RHEL:

```bash
# Install yum-utils
sudo yum install -y yum-utils

# Add Docker repository
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker Engine
sudo yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start and enable Docker
sudo systemctl enable docker
sudo systemctl start docker

# Add your user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

## Verify on Linux:

```bash
docker --version
docker run hello-world
```

🎉 **Docker is installed on Linux!**

---

## ✅ Verification Checklist

```bash
# 1. Docker is installed
docker --version
# Expected: Docker version 24.x.x

# 2. Docker is running
docker ps
# Expected: Empty list (no containers yet, but no error)

# 3. Can run containers
docker run hello-world
# Expected: "Hello from Docker!" message

# 4. Docker info
docker info
# Expected: Lots of system information
```

---

## 🚀 Next Step

After Docker is installed, install kubectl:

**[Install kubectl →](../kubectl/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Docker is running! Your container engine is ready!      ║
║     Time to install kubectl! 🔧                             ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

*Made with ❤️ by Rithu — "Containers are the future!"* 🧑‍🏫
