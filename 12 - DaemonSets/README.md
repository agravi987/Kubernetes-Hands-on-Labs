# Lab 12 — DaemonSets

<div align="center">

![Lab-12](https://img.shields.io/badge/Lab-12-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Medium-purple)
![Time](https://img.shields.io/badge/Time-25%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-DaemonSets-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 12 — DaemonSets                                         ║
║  "One Pod Per Node, No Exceptions"                           ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"A DaemonSet is like a security guard — it makes sure there's exactly one at every entrance. No matter how many nodes you add, a DaemonSet pod will be there!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand DaemonSets and their use cases
- ✅ Deploy a DaemonSet (log collector simulation)
- ✅ Use node selectors with DaemonSets
- ✅ Understand DaemonSet update strategies

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-11
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
│                     DAEMONSET ARCHITECTURE                       │
│                                                                  │
│  Cluster with 3 Nodes:                                          │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  Node 1       │  │  Node 2       │  │  Node 3       │         │
│  │               │  │               │  │               │         │
│  │  ┌─────────┐ │  │  ┌─────────┐ │  │  ┌─────────┐ │         │
│  │  │ Fluentd │ │  │  │ Fluentd │ │  │  │ Fluentd │ │         │
│  │  │ (Daemon │ │  │  │ (Daemon │ │  │  │ (Daemon │ │         │
│  │  │  Set)   │ │  │  │  Set)   │ │  │  │  Set)   │ │         │
│  │  └─────────┘ │  │  └─────────┘ │  │  └─────────┘ │         │
│  │  ┌─────────┐ │  │  ┌─────────┐ │  │  ┌─────────┐ │         │
│  │  │  App    │ │  │  │  App    │ │  │  │  App    │ │         │
│  │  │  Pod    │ │  │  │  Pod    │ │  │  │  Pod    │ │         │
│  │  └─────────┘ │  │  └─────────┘ │  │  └─────────┘ │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                  │
│  New Node 4 joins → DaemonSet auto-deploys Fluentd there!      │
│  ┌──────────────┐                                              │
│  │  Node 4       │                                              │
│  │  ┌─────────┐ │                                              │
│  │  │ Fluentd │ │  ← Automatically scheduled!                  │
│  │  └─────────┘ │                                              │
│  └──────────────┘                                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create a DaemonSet

```bash
cat > fluentd-daemonset.yaml << 'EOF'
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-logger
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.16-debian-elasticsearch8
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: containers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: containers
        hostPath:
          path: /var/lib/docker/containers
EOF
```

```bash
kubectl apply -f fluentd-daemonset.yaml
kubectl get daemonset
```

Expected output:
```
NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd-logger  1         1         1       1            1           <none>          10s
```

```bash
# Check the pod
kubectl get pods -l app=fluentd -o wide
```

Expected output:
```
NAME                  READY   STATUS    RESTARTS   AGE    IP           NODE
fluentd-logger-xxxxx  1/1     Running   0          30s    172.17.0.2   minikube
```

📸 **Screenshot Placeholder:** *[Terminal showing DaemonSet pod on node]*

> 💡 **Rithu's Tip:** *"Since Minikube is single-node, there's only one DaemonSet pod. In a multi-node cluster, you'd see one pod per node!"*

---

### Step 2: Examine the DaemonSet

```bash
kubectl describe daemonset fluentd-logger
```

Key sections:
```
Namespace:          default
Selector:           app=fluentd
Node-Selector:      <none>
Desired Number of Nodes: 1
Number of Nodes with at least 1 DaemonSet Pod Running: 1
Number of Nodes Scheduled to run DaemonSet Pods: 1
```

> 💡 **Rithu's Tip:** *"Notice 'Node-Selector: <none>' — this DaemonSet runs on ALL nodes. We'll add node selectors next!"*

---

### Step 3: Node Selector with DaemonSet

```bash
cat > logging-daemonset-nodeselector.yaml << 'EOF'
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  labels:
    app: log-collector
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      nodeSelector:
        role: logging
      containers:
      - name: log-agent
        image: busybox:latest
        command: ["sh", "-c", "echo 'Log agent running on $(hostname)' && sleep 3600"]
      tolerations:
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
        effect: "NoSchedule"
EOF
```

```bash
kubectl apply -f logging-daemonset-nodeselector.yaml
kubectl get pods -l app=log-collector -o wide
```

On Minikube (single node without the label), the pod might not be scheduled. Let's add the label:

```bash
kubectl label node minikube role=logging
kubectl get pods -l app=log-collector -o wide
```

Now the pod should appear!

> 💡 **Rithu's Tip:** *"Node selectors let you control which nodes run DaemonSet pods. Useful when you only want log collectors on worker nodes, not the control plane!"*

📸 **Screenshot Placeholder:** *[Terminal showing DaemonSet with node selector]*

---

### Step 4: DaemonSet Update Strategy

```bash
kubectl get daemonset fluentd-logger -o yaml | grep -A 10 strategy
```

Default is RollingUpdate with maxUnavailable: 1. This means:
- One pod is updated at a time
- The next pod starts updating only after the current one is ready

Other options:
- **RollingUpdate** (default): Update pods one by one
- **OnDelete**: Only update pods when manually deleted

---

### Step 5: Clean Up

```bash
kubectl delete daemonset fluentd-logger
kubectl delete daemonset log-collector
kubectl label node minikube role-  # Remove the label
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. DaemonSet created
kubectl get daemonset
# Expected: fluentd-logger with 1/1 DESIRED/READY

# 2. Pod on node
kubectl get pods -l app=fluentd -o wide
# Expected: Pod running on minikube node

# 3. Cleanup
kubectl delete daemonset --all
```

---

## 🧹 Cleanup

```bash
kubectl delete daemonset --all
kubectl label node minikube role- 2>/dev/null
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **DaemonSet** | Ensures one pod runs on every (or selected) node |
| **Node Selector** | Filter which nodes run DaemonSet pods |
| **Tolerations** | Allow DaemonSet pods on nodes with taints |
| **Rolling Update** | Update DaemonSet pods one at a time |
| **hostPath** | Access node's filesystem from pods |

---

## 🚀 What's Next?

Let's control where pods are scheduled:

**[Lab 13: Node Affinity & Taints →](../13 - Node Affinity and Taints/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 DaemonSets conquered! Your infrastructure pods          ║
║     are now everywhere they need to be! 🌐                  ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
