# Lab 17 — Network Policies

<div align="center">

![Lab-17](https://img.shields.io/badge/Lab-17-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐⭐%20Hard-red)
![Time](https://img.shields.io/badge/Time-40%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-NetworkPolicy-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 17 — Network Policies                                    ║
║  "Firewall Rules for Your Pods"                               ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"By default, every pod in Kubernetes can talk to every other pod. That's like an apartment building with no locks on any door. Network Policies are those locks!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Create a default deny policy
- ✅ Allow specific ingress traffic
- ✅ Allow specific egress traffic
- ✅ Use pod selectors and namespace selectors

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-16
- [ ] Minikube running with a CNI that supports NetworkPolicy (Calico, Weave)

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   NETWORK POLICY FLOW                            │
│                                                                  │
│  DEFAULT: All traffic allowed (no policies)                     │
│  ┌────────┐ ──────▶ ┌────────┐ ──────▶ ┌────────┐            │
│  │ Pod A  │ ──────▶ │ Pod B  │ ──────▶ │Pod C   │            │
│  └────────┘         └────────┘         └────────┘            │
│                                                                  │
│  WITH DEFAULT DENY: All traffic blocked                        │
│  ┌────────┐ ─X──▶ ┌────────┐ ─X──▶ ┌────────┐               │
│  │ Pod A  │       │ Pod B  │       │Pod C   │               │
│  └────────┘       └────────┘       └────────┘               │
│                                                                  │
│  WITH SELECTIVE ALLOW: A→B allowed, B→C blocked               │
│  ┌────────┐ ──────▶ ┌────────┐ ─X──▶ ┌────────┐             │
│  │ Pod A  │  ✅     │ Pod B  │  ❌   │Pod C   │             │
│  └────────┘         └────────┘         └────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create Test Applications

```bash
cat > test-apps.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: netpol-demo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: netpol-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
  namespace: netpol-demo
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: netpol-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
        tier: api
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database
  namespace: netpol-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
        tier: data
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
EOF
```

```bash
kubectl apply -f test-apps.yaml
kubectl get pods -n netpol-demo
```

Wait for all pods to be Running.

---

### Step 2: Default Deny All Ingress

```bash
cat > default-deny.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: netpol-demo
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF
```

```bash
kubectl apply -f default-deny.yaml
```

Now ALL ingress traffic to all pods in the namespace is blocked!

```bash
# Test: Frontend can't be reached
kubectl run test-client --image=busybox:latest -n netpol-demo --rm -it --restart=Never -- wget -qO- --timeout=3 http://frontend-svc
# Expected: Connection timed out!
```

> 💡 **Rithu's Tip:** *"podSelector: {} means 'select all pods'. Combined with policyTypes: Ingress, this blocks ALL incoming traffic to every pod in the namespace!"*

📸 **Screenshot Placeholder:** *[Terminal showing connection timeout after default deny]*

---

### Step 3: Allow Frontend Ingress from Anywhere

```bash
cat > allow-frontend-ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-ingress
  namespace: netpol-demo
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Ingress
  ingress:
  - from: []           # Empty = from anywhere
    ports:
    - protocol: TCP
      port: 80
EOF
```

```bash
kubectl apply -f allow-frontend-ingress.yaml

# Test: Frontend is now accessible
kubectl run test-client --image=busybox:latest -n netpol-demo --rm -it --restart=Never -- wget -qO- --timeout=3 http://frontend-svc
# Expected: nginx welcome page!
```

> 💡 **Rithu's Tip:** *"Network Policies are ADDITIVE — you can layer multiple policies! The default-deny blocks everything, then this policy allows traffic to frontend on port 80."*

📸 **Screenshot Placeholder:** *[Terminal showing frontend accessible again]*

---

### Step 4: Allow Backend to Reach Database

```bash
cat > allow-backend-to-database.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-database
  namespace: netpol-demo
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 80
EOF
```

```bash
kubectl apply -f allow-backend-to-database.yaml

# Test: Backend CAN reach database
kubectl run test-client --image=busybox:latest -n netpol-demo --rm -it \
  --restart=Never --overrides='{"spec":{"containers":[{"name":"test","image":"busybox","command":["sh","-c","wget -qO- --timeout=3 http://database && echo SUCCESS || echo FAILED"],"env":[{"name":"HOSTNAME","valueFrom":{"fieldRef":{"fieldPath":"metadata.name"}}}]}]}}'

# Test: Frontend CANNOT reach database
kubectl run test-frontend --image=busybox:latest -n netpol-demo --rm -it --restart=Never -- sh -c "wget -qO- --timeout=3 http://database && echo SUCCESS || echo FAILED"
# Expected: FAILED (connection timeout)
```

> 💡 **Rithu's Tip:** *"The podSelector in 'from' only allows pods with app: backend to reach the database. Frontend pods are blocked!"*

📸 **Screenshot Placeholder:** *[Terminal showing backend can reach database, frontend cannot]*

---

### Step 5: Allow Egress to DNS

Don't forget — pods need DNS to resolve service names!

```bash
cat > allow-dns-egress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: netpol-demo
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
EOF
```

```bash
kubectl apply -f allow-dns-egress.yaml
```

> 💡 **Rithu's Tip:** *"If you block egress, pods can't resolve DNS! Always allow UDP/TCP port 53 for DNS resolution, or your services will break."*

---

### Step 6: Comprehensive Policy Example

```bash
cat > full-network-policy.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: full-policy
  namespace: netpol-demo
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 80
  - to: []   # Allow DNS
    ports:
    - protocol: UDP
      port: 53
EOF
```

```bash
kubectl apply -f full-network-policy.yaml
```

This single policy says:
- Backend can receive traffic from Frontend on port 80
- Backend can send traffic to Database on port 80
- Backend can resolve DNS
- Everything else is blocked

---

### Step 7: Clean Up

```bash
kubectl delete namespace netpol-demo
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. Default deny blocks all
kubectl apply -f default-deny.yaml
kubectl run test --image=busybox -n netpol-demo --rm -it --restart=Never -- wget -qO- --timeout=3 http://frontend-svc
# Expected: Timeout

# 2. Selective allow works
kubectl apply -f allow-frontend-ingress.yaml
kubectl run test --image=busybox -n netpol-demo --rm -it --restart=Never -- wget -qO- --timeout=3 http://frontend-svc
# Expected: Works

# 3. Cleanup
kubectl delete namespace netpol-demo
```

---

## 🧹 Cleanup

```bash
kubectl delete namespace netpol-demo
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **NetworkPolicy** | Controls traffic flow between pods |
| **Default Deny** | Block all traffic, then selectively allow |
| **podSelector** | Target specific pods by labels |
| **ingress** | Incoming traffic rules |
| **egress** | Outgoing traffic rules |
| **Additive Policies** | Multiple policies stack together |
| **DNS Egress** | Always allow port 53 for name resolution |

---

## 🚀 What's Next?

Let's enforce pod security standards:

**[Lab 18: Pod Security Standards →](../18 - Pod Security Standards/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Network Policies conquered! Your pods now have          ║
║     proper firewall rules! 🔥🔒                             ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
