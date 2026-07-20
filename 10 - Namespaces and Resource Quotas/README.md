# Lab 10 — Namespaces & Resource Quotas

<div align="center">

![Lab-10](https://img.shields.io/badge/Lab-10-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐%20Medium-blue)
![Time](https://img.shields.io/badge/Time-30%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Namespaces%20%2F%20Quotas-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 10 — Namespaces & Resource Quotas                       ║
║  "Divide and Conquer Your Cluster"                           ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Imagine a shared apartment without walls — everyone's stuff everywhere. That's a cluster without Namespaces. Let's add some walls!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Create and manage Namespaces
- ✅ Set Resource Quotas to limit namespace resources
- ✅ Apply LimitRanges for individual pod constraints
- ✅ Understand multi-tenant isolation

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-09
- [ ] Minikube running

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
```

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                     CLUSTER (Multi-Tenant)                       │
│                                                                   │
│  ┌─────────────────────────┐  ┌─────────────────────────┐      │
│  │   Namespace: dev-team   │  │   Namespace: prod-team  │      │
│  │                          │  │                          │      │
│  │  Resource Quota:         │  │  Resource Quota:         │      │
│  │  - CPU: 4 cores          │  │  - CPU: 8 cores          │      │
│  │  - Memory: 8Gi           │  │  - Memory: 16Gi          │      │
│  │  - Pods: 10              │  │  - Pods: 20              │      │
│  │                          │  │                          │      │
│  │  ┌──────┐ ┌──────┐      │  │  ┌──────┐ ┌──────┐      │      │
│  │  │ Pod  │ │ Pod  │ ...  │  │  │ Pod  │ │ Pod  │ ...  │      │
│  │  └──────┘ └──────┘      │  │  └──────┘ └──────┘      │      │
│  └─────────────────────────┘  └─────────────────────────┘      │
│                                                                   │
│  ┌─────────────────────────┐  ┌─────────────────────────┐      │
│  │   Namespace: staging    │  │   Namespace: monitoring │      │
│  │   (Resource Quota set)  │  │   (Resource Quota set)  │      │
│  └─────────────────────────┘  └─────────────────────────┘      │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: List Default Namespaces

```bash
kubectl get namespaces
```

Expected output:
```
NAME              STATUS   AGE
default           Active   30m
kube-node-lease   Active   30m
kube-public       Active   30m
kube-system       Active   30m
```

> 💡 **Rithu's Tip:** *"default: Where resources go if you don't specify a namespace. kube-system: Kubernetes internals. kube-public: Public cluster info. kube-node-lease: Node heartbeat data."*

---

### Step 2: Create Namespaces

```bash
# Method 1: kubectl create
kubectl create namespace dev-team
kubectl create namespace staging
kubectl create namespace prod-team
```

```bash
# Method 2: YAML manifest
cat > namespaces.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
  labels:
    team: ops
    environment: production
---
apiVersion: v1
kind: Namespace
metadata:
  name: testing
  labels:
    team: qa
    environment: test
EOF
```

```bash
kubectl apply -f namespaces.yaml
kubectl get namespaces
```

Expected output:
```
NAME              STATUS   AGE
default           Active   30m
dev-team          Active   10s
monitoring        Active   5s
prod-team         Active   15s
staging           Active   15s
testing           Active   5s
kube-node-lease   Active   30m
kube-public       Active   30m
kube-system       Active   30m
```

📸 **Screenshot Placeholder:** *[Terminal showing all namespaces]*

---

### Step 3: Deploy to Specific Namespace

```bash
# Deploy to dev-team namespace
kubectl create deployment nginx-dev --image=nginx:1.25 --namespace=dev-team
kubectl create deployment nginx-prod --image=nginx:1.25 --namespace=prod-team
```

```bash
# List pods in all namespaces
kubectl get pods --all-namespaces
```

Expected output:
```
NAMESPACE       NAME                        READY   STATUS    AGE
dev-team        nginx-dev-xxxxxxxxx-xxxxx   1/1     Running   10s
prod-team       nginx-prod-xxxxxxxxx-xxxxx  1/1     Running   10s
kube-system     coredns-xxxxxxxxx-xxxxx     1/1     Running   30m
...
```

```bash
# List pods in a specific namespace
kubectl get pods -n dev-team
```

> 💡 **Rithu's Tip:** *"Use -n <namespace> or --namespace=<namespace> to target a specific namespace. Without it, kubectl uses the 'default' namespace!"*

📸 **Screenshot Placeholder:** *[Terminal showing pods across namespaces]*

---

### Step 4: Create Resource Quotas

```bash
cat > dev-team-quota.yaml << 'EOF'
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-team-quota
  namespace: dev-team
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
    pods: "10"
    services: "5"
    persistentvolumeclaims: "3"
    configmaps: "10"
    secrets: "10"
EOF
```

```bash
kubectl apply -f dev-team-quota.yaml
kubectl get resourcequota -n dev-team
```

Expected output:
```
NAME              AGE   REQUEST                           LIMIT
dev-team-quota    10s   cpu: 0/2, memory: 0/4Gi,         cpu: 0/4, memory: 0/8Gi,
                        pods: 0/10                        pods: 0/10
```

📸 **Screenshot Placeholder:** *[Terminal showing ResourceQuota]*

> 💡 **Rithu's Tip:** *"ResourceQuotas prevent one team from hogging all resources. Without quotas, a runaway deployment could consume the entire cluster!"*

---

### Step 5: Create LimitRanges

LimitRanges set default and max limits for individual pods/containers:

```bash
cat > dev-team-limits.yaml << 'EOF'
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-team-limits
  namespace: dev-team
spec:
  limits:
  - type: Container
    default:
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "2"
      memory: "2Gi"
    min:
      cpu: "50m"
      memory: "64Mi"
  - type: Pod
    max:
      cpu: "4"
      memory: "4Gi"
EOF
```

```bash
kubectl apply -f dev-team-limits.yaml
kubectl get limitrange -n dev-team
```

> 💡 **Rithu's Tip:** *"LimitRanges automatically apply resource constraints to pods that don't specify them. This prevents 'that one developer' who deploys a pod with 16 CPU cores!"*

📸 **Screenshot Placeholder:** *[Terminal showing LimitRange]*

---

### Step 6: Test Resource Quotas

```bash
# Create multiple pods in dev-team
for i in $(seq 1 5); do
  kubectl create deployment test-pod-$i --image=nginx:1.25 -n dev-team
done
```

```bash
# Check quota usage
kubectl get resourcequota -n dev-team
```

```bash
# Try to create more pods than the quota allows
for i in $(seq 1 6); do
  kubectl create deployment over-limit-$i --image=nginx:1.25 -n dev-team 2>&1
done
```

You'll eventually see an error like:
```
Error from server (Forbidden): deployments.apps is forbidden: exceeded quota: dev-team-quota,
requested: pods=1, used: pods=10, limited: pods=10
```

> 💡 **Rithu's Tip:** *"That's the quota in action! It stopped you from creating more pods than the team is allowed. This protects the cluster from resource exhaustion!"*

📸 **Screenshot Placeholder:** *[Terminal showing quota exceeded error]*

---

### Step 7: Set Namespace Labels and Selectors

```bash
# Label namespaces
kubectl label namespace dev-team environment=development cost-center=engineering
kubectl label namespace prod-team environment=production cost-center=engineering

# View labels
kubectl get namespaces --show-labels
```

```bash
# List all resources in dev-team
kubectl get all -n dev-team
```

---

### Step 8: Switch Default Namespace

```bash
# Set your default namespace for kubectl
kubectl config set-context --current --namespace=dev-team

# Now you don't need -n for dev-team
kubectl get pods
# Shows dev-team pods!

# Switch back to default
kubectl config set-context --current --namespace=default
```

> 💡 **Rithu's Tip:** *"Setting your default namespace saves typing but can be dangerous. Always double-check which namespace you're in before deleting things!"*

---

### Step 9: Network Isolation Between Namespaces

Namespaces provide logical isolation, but by default pods can still communicate across namespaces. Let's see:

```bash
# Create a pod in dev-team
kubectl run test-net --image=busybox:latest -n dev-team --rm -it --restart=Never -- nslookup nginx-prod.prod-team.svc.cluster.local
```

Expected output:
```
Name:      nginx-prod.prod-team.svc.cluster.local
Address:   172.17.0.X
```

> 💡 **Rithu's Tip:** *"Pods CAN talk across namespaces by default! To truly isolate them, you need Network Policies (Lab 17). Namespaces are for organization, not security — yet!"*

---

### Step 10: Clean Up

```bash
# Delete deployments
kubectl delete deployment --all -n dev-team
kubectl delete deployment --all -n prod-team

# Delete quotas and limits
kubectl delete -f dev-team-quota.yaml
kubectl delete -f dev-team-limits.yaml

# Delete namespaces (deletes everything inside!)
kubectl delete namespace dev-team
kubectl delete namespace staging
kubectl delete namespace prod-team
kubectl delete namespace monitoring
kubectl delete namespace testing

rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. Namespace creation
kubectl get ns dev-team
# Expected: Active

# 2. ResourceQuota
kubectl get resourcequota -n dev-team
# Expected: Quota limits defined

# 3. LimitRange
kubectl get limitrange -n dev-team
# Expected: Limit range defined

# 4. Quota enforcement
kubectl create deployment test --image=nginx -n dev-team
kubectl get resourcequota -n dev-team
# Expected: Resource usage updated

# 5. Cleanup
kubectl delete namespace dev-team
```

---

## 🧹 Cleanup

```bash
kubectl delete namespace dev-team staging prod-team monitoring testing 2>/dev/null
kubectl delete deployment --all -n default 2>/dev/null
kubectl delete resourcequota --all --all-namespaces 2>/dev/null
kubectl delete limitrange --all --all-namespaces 2>/dev/null
rm *.yaml 2>/dev/null
kubectl config set-context --current --namespace=default
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Namespace** | Virtual cluster within a physical cluster |
| **ResourceQuota** | Limit aggregate resource usage per namespace |
| **LimitRange** | Set default and max resource limits per pod/container |
| **Multi-tenancy** | Isolate teams/projects within a shared cluster |
| **Namespace Labels** | Organize and select namespaces |

---

## 🚀 What's Next?

You've completed Section 2! Time for batch jobs and workloads:

**[Lab 11: Jobs & CronJobs →](../11 - Jobs and CronJobs/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Section 2 Complete! You now know how to:                ║
║     Configure apps, manage storage, and organize clusters!  ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
