# Lab 16 — RBAC

<div align="center">

![Lab-16](https://img.shields.io/badge/Lab-16-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Medium-purple)
![Time](https://img.shields.io/badge/Time-35%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-RBAC-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 16 — RBAC (Role-Based Access Control)                   ║
║  "Who Can Do What to Whom?"                                  ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"RBAC is like the bouncer at a VIP club — it checks your ID (ServiceAccount), sees what VIP level you have (Role), and decides if you can enter (RoleBinding). No VIP? No entry!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Create ServiceAccounts
- ✅ Define Roles and ClusterRoles
- ✅ Bind roles with RoleBinding and ClusterRoleBinding
- ✅ Test access with kubectl auth can-i

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-15
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
│                      RBAC COMPONENTS                             │
│                                                                  │
│  ┌─────────────────┐                                           │
│  │  ServiceAccount  │  ← Identity for a pod/process             │
│  │  (who)           │                                           │
│  └────────┬────────┘                                           │
│           │                                                      │
│           │  RoleBinding (namespace scope)                      │
│           │  ClusterRoleBinding (cluster scope)                 │
│           │                                                      │
│           ▼                                                      │
│  ┌─────────────────┐                                           │
│  │  Role            │  ← What can be done                       │
│  │  (what)          │     (verbs + resources + apiGroups)       │
│  └────────┬────────┘                                           │
│           │                                                      │
│           ▼                                                      │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  apiGroups: [""]                                       │      │
│  │  resources: ["pods", "services"]                      │      │
│  │  verbs: ["get", "list", "watch"]                      │      │
│  │  → "You can only watch pods and services"             │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                  │
│  SCOPE:                                                          │
│  ├── Role: Namespace-specific                                   │
│  └── ClusterRole: Cluster-wide                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create a ServiceAccount

```bash
cat > rbac-setup.yaml << 'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-viewer
  namespace: default
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-manager
  namespace: default
EOF
```

```bash
kubectl apply -f rbac-setup.yaml
kubectl get serviceaccounts
```

Expected output:
```
NAME              SECRETS   AGE
default           0         30m
pod-manager       0         10s
pod-viewer        0         10s
```

> 💡 **Rithu's Tip:** *"ServiceAccounts are the 'who' in RBAC. Every pod has a ServiceAccount — the 'default' one if you don't specify one."*

---

### Step 2: Create a Role (Read-Only)

```bash
cat > read-only-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list"]
EOF
```

```bash
kubectl apply -f read-only-role.yaml
kubectl get roles
```

Expected output:
```
NAME         CREATED
pod-reader   10s
```

```bash
# Describe the role
kubectl describe role pod-reader
```

Expected output:
```
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods/log   []                 []              [get list watch]
  pods       []                 []              [get list watch]
  services   []                 []              [get list]
```

> 💡 **Rithu's Tip:** *"The 'rules' section is where the magic happens. We're saying: you can get, list, and watch pods; you can get and list services. That's it!"*

📸 **Screenshot Placeholder:** *[Terminal showing Role definition]*

---

### Step 3: Create a ClusterRole (Cluster-Wide)

```bash
cat > cluster-reader-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: [""]
  resources: ["nodes", "namespaces"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["get", "list", "watch"]
EOF
```

```bash
kubectl apply -f cluster-reader-role.yaml
kubectl get clusterroles | grep cluster-reader
```

> 💡 **Rithu's Tip:** *"ClusterRoles have no namespace — they apply cluster-wide. Perfect for reading nodes, namespaces, or cluster-level resources!"*

---

### Step 4: Bind Role to ServiceAccount (RoleBinding)

```bash
cat > role-bindings.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-viewer-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: pod-viewer
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-viewer-binding
subjects:
- kind: ServiceAccount
  name: pod-viewer
  namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
EOF
```

```bash
kubectl apply -f role-bindings.yaml
kubectl get rolebindings
kubectl get clusterrolebindings | grep pod-viewer
```

> 💡 **Rithu's Tip:** *"RoleBinding attaches a Role to a subject (ServiceAccount, User, or Group). Think of it as granting someone a specific permission set!"*

📸 **Screenshot Placeholder:** *[Terminal showing RoleBindings]*

---

### Step 5: Create Admin Role

```bash
cat > admin-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-admin
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["*"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-manager-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: pod-manager
  namespace: default
roleRef:
  kind: Role
  name: pod-admin
  apiGroup: rbac.authorization.k8s.io
EOF
```

```bash
kubectl apply -f admin-role.yaml
```

---

### Step 6: Test Access with `kubectl auth can-i`

```bash
# Test pod-viewer permissions
kubectl auth can-i get pods --as=system:serviceaccount:default:pod-viewer
# Expected: yes

kubectl auth can-i delete pods --as=system:serviceaccount:default:pod-viewer
# Expected: no

kubectl auth can-i create deployments --as=system:serviceaccount:default:pod-viewer
# Expected: no
```

```bash
# Test pod-manager permissions
kubectl auth can-i delete pods --as=system:serviceaccount:default:pod-manager
# Expected: yes

kubectl auth can-i create deployments --as=system:serviceaccount:default:pod-manager
# Expected: yes
```

```bash
# List all permissions for a ServiceAccount
kubectl auth can-i --list --as=system:serviceaccount:default:pod-viewer
```

> 💡 **Rithu's Tip:** *"kubectl auth can-i is your debugging best friend for RBAC! Use it to verify permissions before deploying applications!"*

📸 **Screenshot Placeholder:** *[Terminal showing auth can-i results]*

---

### Step 7: Use ServiceAccount in a Pod

```bash
cat > sa-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: viewer-pod
spec:
  serviceAccountName: pod-viewer
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["sleep", "3600"]
EOF
```

```bash
kubectl apply -f sa-pod.yaml
kubectl get pods viewer-pod
```

```bash
# Test permissions from inside the pod
kubectl exec viewer-pod -- kubectl get pods
# Expected: Works!

kubectl exec viewer-pod -- kubectl delete pod nginx
# Expected: Error — not allowed!
```

> 💡 **Rithu's Tip:** *"The pod now uses the 'pod-viewer' ServiceAccount instead of 'default'. Its permissions are exactly what we defined in the Role!"*

---

### Step 8: Clean Up

```bash
kubectl delete -f .
kubectl delete rolebinding pod-viewer-binding pod-manager-binding
kubectl delete clusterrolebinding cluster-viewer-binding
kubectl delete role pod-reader pod-admin
kubectl delete clusterrole cluster-reader
kubectl delete serviceaccount pod-viewer pod-manager
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. ServiceAccount exists
kubectl get sa pod-viewer
# Expected: pod-viewer exists

# 2. Role exists
kubectl get role pod-reader
# Expected: pod-reader exists

# 3. Permission check
kubectl auth can-i get pods --as=system:serviceaccount:default:pod-viewer
# Expected: yes

# 4. Cleanup
kubectl delete sa,role,rolebinding,clusterrole,clusterrolebinding --all 2>/dev/null
```

---

## 🧹 Cleanup

```bash
kubectl delete sa --all 2>/dev/null
kubectl delete role --all 2>/dev/null
kubectl delete rolebinding --all 2>/dev/null
kubectl delete clusterrole pod-viewer pod-manager 2>/dev/null
kubectl delete clusterrolebinding --all 2>/dev/null
kubectl delete pod --all
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **ServiceAccount** | Identity for pods and processes |
| **Role** | Defines permissions within a namespace |
| **ClusterRole** | Defines permissions cluster-wide |
| **RoleBinding** | Attaches a Role to a subject |
| **ClusterRoleBinding** | Attaches a ClusterRole to a subject |
| **Verbs** | get, list, watch, create, update, delete, * |
| **can-i** | Test if a subject has a specific permission |

---

## 🚀 What's Next?

Let's add network-level security:

**[Lab 17: Network Policies →](../17 - Network Policies/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 RBAC mastered! Your cluster now has proper access       ║
║     control — no more "everyone is admin"! 🔐              ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
