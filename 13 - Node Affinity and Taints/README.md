# Lab 13 — Node Affinity & Taints/Tolerations

<div align="center">

![Lab-13](https://img.shields.io/badge/Lab-13-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Medium-purple)
![Time](https://img.shields.io/badge/Time-30%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Scheduling-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 13 — Node Affinity & Taints/Tolerations                  ║
║  "Kubernetes Matchmaker: Pods Meet Nodes"                     ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Scheduling in Kubernetes is like a dating app — pods have preferences (affinity) and nodes have deal-breakers (taints). Let's make the perfect match!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Use node affinity to schedule pods on specific nodes
- ✅ Apply taints to nodes
- ✅ Configure tolerations for pods
- ✅ Understand preferential vs required scheduling

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-12
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
│                  SCHEDULING MECHANISMS                           │
│                                                                  │
│  NODE AFFINITY (Pod → Node: "I want to go there")              │
│  ┌──────────┐         ┌──────────┐                             │
│  │   Pod     │ ──────▶│  Node A  │  preferredDuring...        │
│  │ "I prefer │         │ (GPU)    │  "I'd like GPU nodes"     │
│  │  GPU!"   │         └──────────┘                             │
│  └──────────┘                                                   │
│                                                                  │
│  TAINTS (Node → Pod: "Stay away!")                              │
│  ┌──────────┐         ┌──────────┐                             │
│  │   Pod     │ ──X──▶ │  Node B  │  taint: "dedicated=teamA" │
│  │           │         │ (Tainted)│  Pod can't schedule!      │
│  └──────────┘         └──────────┘                             │
│                                                                  │
│  TOLERATIONS (Pod: "I can handle that taint!")                  │
│  ┌──────────┐         ┌──────────┐                             │
│  │   Pod     │ ──────▶│  Node B  │  toleration: "teamA"      │
│  │ "I can   │         │ (Tainted)│  Pod CAN schedule!        │
│  │  handle   │         └──────────┘                             │
│  │  that!"  │                                                   │
│  └──────────┘                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Node Affinity — Preferred

```bash
cat > preferred-affinity.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: preferred-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: disk-type
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx:1.25
EOF
```

```bash
# Label the node
kubectl label node minikube disk-type=ssd

# Apply the pod
kubectl apply -f preferred-affinity.yaml
kubectl get pods preferred-pod -o wide
```

> 💡 **Rithu's Tip:** *"preferredDuringScheduling means 'try to schedule here, but if not possible, put it anywhere.' The weight (80) is the priority — higher weight = stronger preference!"*

---

### Step 2: Node Affinity — Required

```bash
cat > required-affinity.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: required-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk-type
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx:1.25
EOF
```

```bash
kubectl apply -f required-affinity.yaml
kubectl get pods required-pod -o wide
```

> 💡 **Rithu's Tip:** *"requiredDuringScheduling means 'schedule ONLY on matching nodes.' If no node matches, the pod stays Pending!"*

📸 **Screenshot Placeholder:** *[Terminal showing pod scheduled on labeled node]*

---

### Step 3: Apply Taints to Nodes

```bash
# Taint the node — only pods with matching toleration can schedule here
kubectl taint nodes minikube dedicated=teamA:NoSchedule
```

```bash
# Check the taint
kubectl describe node minikube | grep -A 5 Taints
```

Expected output:
```
Taints:             dedicated=teamA:NoSchedule
```

Now try creating a pod WITHOUT toleration:

```bash
cat > no-toleration-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: blocked-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
EOF
```

```bash
kubectl apply -f no-toleration-pod.yaml
kubectl get pods blocked-pod
```

Expected output:
```
NAME          READY   STATUS    RESTARTS   AGE
blocked-pod   0/1     Pending   0          30s
```

> 💡 **Rithu's Tip:** *"The pod is Pending because the only node (minikube) has a taint that this pod doesn't tolerate!"*

---

### Step 4: Add Toleration

```bash
cat > with-toleration-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: tolerated-pod
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "teamA"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx:1.25
EOF
```

```bash
kubectl apply -f with-toleration-pod.yaml
kubectl get pods -o wide
```

Expected output:
```
NAME             READY   STATUS    RESTARTS   AGE
blocked-pod      0/1     Pending   0          2m
tolerated-pod    1/1     Running   0          10s
```

> 💡 **Rithu's Tip:** *"Tolerations say 'I can handle that taint.' They DON'T attract pods to tainted nodes — they just allow pods to be scheduled there!"*

📸 **Screenshot Placeholder:** *[Terminal showing one pod pending, one running]*

---

### Step 5: Taint Effects

Different taint effects:

| Effect | Behavior |
|--------|----------|
| **NoSchedule** | New pods won't schedule, existing pods stay |
| **PreferNoSchedule** | Scheduler tries to avoid, but may schedule if needed |
| **NoExecute** | New pods won't schedule AND existing pods are evicted |

```bash
# Remove the current taint
kubectl taint nodes minikube dedicated=teamA:NoSchedule-

# Add a NoExecute taint
kubectl taint nodes minikube critical=only-for-system:NoExecute
```

> 💡 **Rithu's Tip:** *"NoExecute is powerful — it kicks out existing pods that don't tolerate the new taint! Use it carefully!"*

```bash
# Remove the taint
kubectl taint nodes minikube critical=only-for-system:NoExecute-
```

---

### Step 6: Combine Affinity + Taints

```bash
cat > combined-scheduling.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: gpu-app
  template:
    metadata:
      labels:
        app: gpu-app
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 90
            preference:
              matchExpressions:
              - key: gpu
                operator: In
                values:
                - "true"
      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "teamA"
        effect: "NoSchedule"
      containers:
      - name: app
        image: busybox:latest
        command: ["sh", "-c", "echo 'Running on node $(hostname)' && sleep 3600"]
EOF
```

```bash
kubectl apply -f combined-scheduling.yaml
```

---

### Step 7: Clean Up

```bash
kubectl delete -f .
kubectl taint nodes minikube dedicated=teamA:NoSchedule- 2>/dev/null
kubectl taint nodes minikube critical=only-for-system:NoExecute- 2>/dev/null
kubectl label node minikube disk-type- 2>/dev/null
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. Node labels
kubectl label node minikube disk-type=ssd
kubectl get node minikube --show-labels

# 2. Taints
kubectl taint nodes minikube dedicated=teamA:NoSchedule
kubectl describe node minikube | grep Taints

# 3. Toleration allows scheduling
kubectl apply -f with-toleration-pod.yaml
kubectl get pod tolerated-pod
# Expected: Running

# 4. Cleanup
kubectl delete pod --all
kubectl taint nodes minikube dedicated=teamA:NoSchedule-
kubectl label node minikube disk-type-
```

---

## 🧹 Cleanup

```bash
kubectl delete pod --all
kubectl delete deploy --all
kubectl taint nodes minikube dedicated=teamA:NoSchedule- 2>/dev/null
kubectl label node minikube disk-type- 2>/dev/null
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Node Affinity** | Pod preference for certain nodes |
| **Required Affinity** | Hard requirement — must match or stay Pending |
| **Preferred Affinity** | Soft preference — try to match, but schedule anywhere |
| **Taints** | Node restrictions that repel pods |
| **Tolerations** | Pod allowance to schedule on tainted nodes |
| **NoSchedule** | Prevents new pods from scheduling |
| **NoExecute** | Evicts existing pods AND prevents new ones |

---

## 🚀 What's Next?

Let's make pods scale automatically based on load:

**[Lab 14: Horizontal Pod Autoscaling →](../14 - Horizontal Pod Autoscaling/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Pods and nodes are now perfectly matched!               ║
║     The Kubernetes matchmaker strikes again! 💕             ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
