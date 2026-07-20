# Lab 15 — Pod Disruption Budgets

<div align="center">

![Lab-15](https://img.shields.io/badge/Lab-15-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Medium-purple)
![Time](https://img.shields.io/badge/Time-25%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-PDB-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 15 — Pod Disruption Budgets                             ║
║  "Stay Available During Maintenance"                         ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"You're upgrading your servers and accidentally take down ALL replicas. Ouch! PDBs make sure you always have enough pods running, even during maintenance."* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand voluntary vs involuntary disruptions
- ✅ Create PodDisruptionBudgets
- ✅ Test PDB protection during node drains
- ✅ Configure minAvailable vs maxUnavailable

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-14
- [ ] Minikube running

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  DISRUPTION TYPES                                │
│                                                                  │
│  VOLUNTARY DISRUPTIONS (PDB protects against these):           │
│  ├── kubectl drain                                              │
│  ├── kubectl delete pod                                         │
│  ├── Pod eviction (resource pressure)                           │
│  └── Node upgrade/maintenance                                   │
│                                                                  │
│  INVOLUNTARY DISRUPTIONS (PDB cannot protect):                  │
│  ├── Hardware failure                                           │
│  ├── Kernel panic                                               │
│  ├── VM termination (cloud provider)                            │
│  └── Out of memory kills                                        │
│                                                                  │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  PDB: "minAvailable: 2"                               │      │
│  │                                                        │      │
│  │  Before drain: [Pod1] [Pod2] [Pod3] [Pod4] [Pod5]   │      │
│  │  Drain attempt: Can remove at most 3 (5-2=3)         │      │
│  │                                                        │      │
│  │  After drain:   [Pod1] [Pod2] ← Always at least 2!   │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create a Deployment

```bash
cat > web-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
EOF
```

```bash
kubectl apply -f web-deployment.yaml
kubectl get pods -l app=web-server -w
```

Wait for all 5 pods to be Running.

---

### Step 2: Create a PDB (minAvailable)

```bash
cat > pdb-minavailable.yaml << 'EOF'
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-server-pdb
spec:
  minAvailable: 3
  selector:
    matchLabels:
      app: web-server
EOF
```

```bash
kubectl apply -f pdb-minavailable.yaml
kubectl get pdb
```

Expected output:
```
NAME           MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
web-server-pdb 3               <none>            2                     10s
```

> 💡 **Rithu's Tip:** *"ALLOWED DISRUPTIONS: 2 means you can safely disrupt 2 pods at a time while maintaining 3 available!"*

📸 **Screenshot Placeholder:** *[Terminal showing PDB with 5 pods and minAvailable of 3]*

---

### Step 3: Test PDB with kubectl drain

```bash
# Try to drain the node (simulates maintenance)
kubectl drain minikube --ignore-daemonsets --delete-emptydir-data --force
```

You might see an error or warning about PDB violations! This is the PDB protecting your pods.

```bash
# Check how many pods remain
kubectl get pods -l app=web-server
```

---

### Step 4: Create PDB with maxUnavailable

```bash
# First, uncordon to bring node back
kubectl uncordon minikube
```

```bash
cat > pdb-maxunavailable.yaml << 'EOF'
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-server-pdb
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: web-server
EOF
```

```bash
kubectl delete pdb web-server-pdb
kubectl apply -f pdb-maxunavailable.yaml
kubectl get pdb
```

Expected output:
```
NAME           MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
web-server-pdb <none>          1                 4                     10s
```

> 💡 **Rithu's Tip:** *"maxUnavailable: 1 means at most 1 pod can be disrupted at a time. With 5 pods, you could disrupt 4 (5-1=4) but only 1 at a time!"*

---

### Step 5: Test PDB Protection

```bash
# Try to delete multiple pods — PDB should block it
kubectl delete pod -l app=web-server --field-selector=status.phase=Running
```

The PDB will prevent all pods from being deleted simultaneously.

```bash
# Check remaining pods
kubectl get pods -l app=web-server
```

You'll see some pods survived!

---

### Step 6: PDB with Percentage

```bash
cat > pdb-percent.yaml << 'EOF'
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-server-pdb-pct
spec:
  minAvailable: "60%"
  selector:
    matchLabels:
      app: web-server
EOF
```

```bash
kubectl delete pdb web-server-pdb
kubectl apply -f pdb-percent.yaml
kubectl get pdb
```

> 💡 **Rithu's Tip:** *"Percentage-based PDBs are great for scaling deployments. 60% of 5 pods = 3, 60% of 10 pods = 6. Always scales with your deployment!"*

---

### Step 7: Clean Up

```bash
kubectl delete pdb --all
kubectl delete -f web-deployment.yaml
kubectl uncordon minikube 2>/dev/null
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. PDB created
kubectl get pdb
# Expected: PDB exists with correct rules

# 2. PDB protection works
# (Attempt drain and verify pods remain)

# 3. Cleanup
kubectl delete pdb --all
kubectl delete deploy --all
```

---

## 🧹 Cleanup

```bash
kubectl delete pdb --all
kubectl delete deploy --all
kubectl uncordon minikube 2>/dev/null
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **PodDisruptionBudget** | Limits voluntary disruptions to maintain availability |
| **minAvailable** | Minimum pods that must remain running |
| **maxUnavailable** | Maximum pods that can be unavailable |
| **Allowed Disruptions** | How many pods can be disrupted right now |
| **Voluntary Disruptions** | Drain, delete, upgrade — PDB protects against these |
| **Involuntary Disruptions** | Hardware failure, OOM — PDB can't help |

---

## 🚀 What's Next?

You've completed Section 3! Time for security:

**[Lab 16: RBAC →](../16 - RBAC/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Section 3 Complete! Your workloads are now:             ║
║     Scheduled, scaled, and disruption-proof! 💪             ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
