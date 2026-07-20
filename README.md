# 🚀 Kubernetes Hands-On Labs

<div align="center">

![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.28+-326CE5?logo=kubernetes&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner%20to%20Advanced-green)
![Labs](https://img.shields.io/badge/Labs-25-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Contributor](https://img.shields.io/badge/Contributor-Rithu-blue)

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║   🎓  K U B E R N E T E S   H A N D S - O N   L A B S     ║
║                                                              ║
║      From Zero to Cluster Hero — One Lab at a Time!         ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

</div>

---

> *"Kubernetes: because manually managing 500 servers at 3 AM is so last decade."* — **Rithu** 🧑‍🏫

---

## 👋 Welcome, Future Kubernetes Hero!

Hey there! I'm **Rithu**, and I'll be your guide on this epic journey through the world of Kubernetes. Whether you're a developer who's tired of the classic "it works on my machine" excuse, or an ops engineer who wants to sleep through the night without PagerDuty screaming at you — you're in the right place.

This repository is a **comprehensive, hands-on lab series** that takes you from *"What's a pod?"* to confidently deploying production-grade microservices on Kubernetes.

**Ravi**, our curious student, asked me to build something that's:
- 📚 **Educational** — not just copy-paste, but actually *understand* what you're doing
- 🛠️ **Hands-On** — every concept has a lab you can follow along
- 🎯 **Progressive** — starts easy, builds up to real-world scenarios
- 💡 **Fun** — because learning shouldn't feel like a chore

So here it is, Ravi. Let's go! 🚀

---

## 📋 Lab Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    LEARNING PATH OVERVIEW                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Section 1: Fundamentals (Labs 01-05)                          │
│  ├── Lab 01: Minikube & kubectl Setup           ⭐ Beginner    │
│  ├── Lab 02: Pods Deep Dive                     ⭐ Beginner    │
│  ├── Lab 03: ReplicaSets                        ⭐ Beginner    │
│  ├── Lab 04: Deployments                        ⭐⭐ Easy-Med  │
│  └── Lab 05: Services & Networking              ⭐⭐ Easy-Med  │
│                                                                 │
│  Section 2: Configuration & Storage (Labs 06-10)               │
│  ├── Lab 06: ConfigMaps & Secrets               ⭐⭐ Easy-Med  │
│  ├── Lab 07: Persistent Volumes & Claims        ⭐⭐ Medium    │
│  ├── Lab 08: StatefulSets                       ⭐⭐⭐ Medium   │
│  ├── Lab 09: Ingress Controller                 ⭐⭐⭐ Medium   │
│  └── Lab 10: Namespaces & Resource Quotas       ⭐⭐ Medium    │
│                                                                 │
│  Section 3: Workloads & Scheduling (Labs 11-15)                │
│  ├── Lab 11: Jobs & CronJobs                    ⭐⭐ Medium    │
│  ├── Lab 12: DaemonSets                         ⭐⭐⭐ Medium   │
│  ├── Lab 13: Node Affinity & Taints             ⭐⭐⭐ Medium   │
│  ├── Lab 14: Horizontal Pod Autoscaling         ⭐⭐⭐ Med-Hard│
│  └── Lab 15: Pod Disruption Budgets             ⭐⭐⭐ Medium   │
│                                                                 │
│  Section 4: Security (Labs 16-19)                               │
│  ├── Lab 16: RBAC                               ⭐⭐⭐ Medium   │
│  ├── Lab 17: Network Policies                   ⭐⭐⭐⭐ Hard   │
│  ├── Lab 18: Pod Security Standards             ⭐⭐⭐ Medium   │
│  └── Lab 19: Service Mesh (Istio Basics)        ⭐⭐⭐⭐ Hard   │
│                                                                 │
│  Section 5: Production & Ops (Labs 20-23)                       │
│  ├── Lab 20: Helm Charts                        ⭐⭐⭐ Medium   │
│  ├── Lab 21: Monitoring with Prometheus/Grafana ⭐⭐⭐⭐ Hard   │
│  ├── Lab 22: Logging with EFK Stack             ⭐⭐⭐⭐ Hard   │
│  └── Lab 23: Backup with Velero                 ⭐⭐⭐ Med-Hard│
│                                                                 │
│  Section 6: Cloud & Capstone (Labs 24-25)                       │
│  ├── Lab 24: EKS Cluster Setup                  ⭐⭐⭐⭐ Hard   │
│  └── Lab 25: Capstone - Microservices           ⭐⭐⭐⭐⭐ Expert│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🗺️ What You'll Learn

| Section | Skills Unlocked |
|---------|----------------|
| **Fundamentals** | Pods, ReplicaSets, Deployments, Services, kubectl mastery |
| **Config & Storage** | ConfigMaps, Secrets, PV/PVC, StatefulSets, Ingress, Namespaces |
| **Workloads** | Jobs, CronJobs, DaemonSets, Affinity, Autoscaling, PDBs |
| **Security** | RBAC, Network Policies, Pod Security, Service Mesh |
| **Production** | Helm, Prometheus, Grafana, EFK, Velero |
| **Cloud & Capstone** | EKS, Multi-service architectures, Full production stack |

---

## 🚀 Quick Start

```bash
# 1. Clone this repository
git clone https://github.com/your-username/Kubernetes-Hands-On-Labs.git
cd Kubernetes-Hands-On-Labs

# 2. Install prerequisites (Docker + kubectl)
cat installation/README.md

# 3. Install your Kubernetes distribution
cd installation/minikube   # For Minikube (recommended)
# OR
cd installation/kind        # For kind (fastest)
# OR
cd installation/kubeadm     # For kubeadm (production-like)

# 4. Start with Lab 01
cd "../../01 - Minikube and kubectl Setup"
cat README.md
```

> *"The best way to learn Kubernetes is to break things in a sandbox and then fix them. That's exactly what these labs let you do."* — **Rithu** 🧑‍🏫

---

## 📁 Repository Structure

```
Kubernetes-Hands-On-Labs/
│
├── README.md                           ← You are here!
├── PREREQUISITES.md                    ← Setup guide
│
├── installation/                       ← 🔧 INSTALL YOUR TOOLS FIRST!
│   ├── README.md                       ← Installation Hub (start here!)
│   ├── docker/                         ← Docker installation guide
│   ├── kubectl/                        ← kubectl installation guide
│   ├── minikube/                       ← Minikube (Windows/Mac/Linux)
│   ├── kind/                           ← kind (Windows/Mac/Linux)
│   └── kubeadm/                        ← kubeadm (production-like)
│
├── 01 - Minikube and kubectl Setup/
│   └── README.md
├── 02 - Pods Deep Dive/
│   └── README.md
├── 03 - ReplicaSets/
│   └── README.md
├── ...
│
├── 25 - Capstone Microservices/
│   ├── README.md
│   ├── frontend/
│   ├── api/
│   └── helm-chart/
│
└── images/                             ← Screenshots & diagrams
```

---

## 🎓 Learning Path Recommendation

```
                    ┌──────────────┐
                    │  Complete    │
                    │  Section 1   │
                    │  (Labs 01-05)│
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │  Complete    │
                    │  Section 2   │
                    │ (Labs 06-10) │
                    └──────┬───────┘
                           │
                ┌──────────┴──────────┐
                │                     │
         ┌──────▼───────┐    ┌───────▼──────┐
         │  Section 3   │    │  Section 4    │
         │ (Labs 11-15) │    │ (Labs 16-19) │
         └──────┬───────┘    └───────┬──────┘
                │                     │
                └──────────┬──────────┘
                           │
                    ┌──────▼───────┐
                    │  Complete    │
                    │  Section 5   │
                    │ (Labs 20-23) │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │  CAPSTONE!   │
                    │ (Labs 24-25) │
                    └──────────────┘
```

> *"Don't skip ahead! Each lab builds on the previous one. Think of it like a Netflix series — you can't watch the finale without the character development."* — **Rithu** 🧑‍🏫

---

## 🤝 Contributing

Found a bug? Have an improvement? Want to add a new lab?

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-lab`)
3. Commit your changes (`git commit -m 'Add amazing lab'`)
4. Push to the branch (`git push origin feature/amazing-lab`)
5. Open a Pull Request

---

## 📜 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- The amazing [Kubernetes community](https://kubernetes.io/community/)
- [Ravi](https://github.com/ravi) — for being the curious student who inspired this
- Everyone who contributes to making Kubernetes less intimidating

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║   Ready to start your Kubernetes journey?                    ║
║                                                              ║
║   👉 Head to Lab 01 and let's get that cluster running!     ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

**[Start Lab 01 →](01 - Minikube and kubectl Setup/README.md)**

---

*Made with ❤️ by Rithu, for Ravi, and for every developer who ever said "but it works on my machine"*

</div>
