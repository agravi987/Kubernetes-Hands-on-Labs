# Lab 18 — Pod Security Standards

<div align="center">

![Lab-18](https://img.shields.io/badge/Lab-18-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Medium-purple)
![Time](https://img.shields.io/badge/Time-30%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Pod%20Security-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 18 — Pod Security Standards                             ║
║  "Lock Down What Pods Can Do"                                ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Running containers as root is like giving every tenant the master key to the entire building. Pod Security Standards say 'nice try, but no!'"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand Pod Security Admission (PSA)
- ✅ Apply Privileged, Baseline, and Restricted policies
- ✅ See enforcement in action
- ✅ Configure namespace labels for security levels

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-17
- [ ] Kubernetes 1.25+ (Minikube latest)

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  POD SECURITY STANDARDS                          │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  PRIVILEGED (least restrictive)                           │  │
│  │  ├── Full host access                                    │  │
│  │  ├── All capabilities allowed                            │  │
│  │  └── Use case: system-level pods (kube-system)           │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  BASELINE (moderate)                                      │  │
│  │  ├── No privileged containers                            │  │
│  │  ├── No host network/process/IPC                         │  │
│  │  ├── No dangerous capabilities (NET_ADMIN, SYS_ADMIN)   │  │
│  │  └── Use case: most application workloads                │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  RESTRICTED (most restrictive)                            │  │
│  │  ├── Non-root user required                              │  │
│  │  ├── Drop all capabilities                               │  │
│  │  ├── Read-only root filesystem                           │  │
│  │  ├── No privilege escalation                             │  │
│  │  └── Use case: security-sensitive workloads              │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ENFORCEMENT MODES:                                              │
│  ├── enforce: Reject if policy violated                        │
│  ├── audit: Log violation but allow                            │
│  └── warn: Show warning but allow                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Understand Pod Security Admission

Pod Security Admission (PSA) replaced PodSecurityPolicy in Kubernetes 1.25+. It works via namespace labels.

```bash
# Check current namespace labels
kubectl describe namespace default
```

---

### Step 2: Create Baseline Namespace

```bash
cat > baseline-namespace.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: baseline-ns
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/warn: restricted
EOF
```

```bash
kubectl apply -f baseline-namespace.yaml
```

> 💡 **Rithu's Tip:** *"enforce: baseline means pods violating baseline will be REJECTED. warn: restricted means restricted violations show warnings but are allowed!"*

---

### Step 3: Test with Privileged Pod (Should Fail on Baseline)

```bash
cat > privileged-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
  namespace: baseline-ns
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    securityContext:
      privileged: true
EOF
```

```bash
kubectl apply -f privileged-pod.yaml
```

Expected output:
```
Error from server (Forbidden): pods "privileged-pod" is forbidden: 
violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
```

> 💡 **Rithu's Tip:** *"The pod was REJECTED because privileged containers are not allowed in baseline policy!"*

📸 **Screenshot Placeholder:** *[Terminal showing pod rejected by baseline policy]*

---

### Step 4: Test with Allowed Pod

```bash
cat > safe-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: safe-pod
  namespace: baseline-ns
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
EOF
```

```bash
kubectl apply -f safe-pod.yaml
kubectl get pods -n baseline-ns
```

Expected output:
```
NAME       READY   STATUS    RESTARTS   AGE
safe-pod   1/1     Running   0          10s
```

---

### Step 5: Restricted Namespace

```bash
cat > restricted-namespace.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: restricted-ns
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
EOF
```

```bash
kubectl apply -f restricted-namespace.yaml
```

Try creating a pod without the restricted requirements:

```bash
cat > basic-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: basic-pod
  namespace: restricted-ns
spec:
  containers:
  - name: nginx
    image: nginx:1.25
EOF
```

```bash
kubectl apply -f basic-pod.yaml
```

Expected output:
```
Error from server (Forbidden): pods "basic-pod" is forbidden: 
violates PodSecurity "restricted:latest": unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"])
runAsNonRoot != true (container "nginx" must not set securityContext.runAsNonRoot=false)
```

---

### Step 6: Create a Restricted-Compliant Pod

```bash
cat > restricted-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: restricted-pod
  namespace: restricted-ns
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
  containers:
  - name: nginx
    image: nginx:1.25
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: nginx-cache
      mountPath: /var/cache/nginx
    - name: nginx-run
      mountPath: /var/run
  volumes:
  - name: tmp
    emptyDir: {}
  - name: nginx-cache
    emptyDir: {}
  - name: nginx-run
    emptyDir: {}
EOF
```

```bash
kubectl apply -f restricted-pod.yaml
kubectl get pods -n restricted-ns
```

Expected output:
```
NAME             READY   STATUS    RESTARTS   AGE
restricted-pod   1/1     Running   0          10s
```

> 💡 **Rithu's Tip:** *"Restricted pods need: non-root user, read-only root filesystem, drop ALL capabilities, no privilege escalation. That's a lot of security wins!"*

📸 **Screenshot Placeholder:** *[Terminal showing restricted pod running]*

---

### Step 7: Audit and Warn Mode

```bash
cat > warn-mode-ns.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: warn-mode-ns
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
EOF
```

```bash
kubectl apply -f warn-mode-ns.yaml
kubectl apply -f basic-pod.yaml -n warn-mode-ns 2>&1
```

You'll see warnings but the pod might still be created!

---

### Step 8: Clean Up

```bash
kubectl delete namespace baseline-ns restricted-ns warn-mode-ns
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. Baseline blocks privileged
kubectl apply -f baseline-namespace.yaml
kubectl apply -f privileged-pod.yaml 2>&1
# Expected: Forbidden error

# 2. Restricted blocks basic pods
kubectl apply -f restricted-namespace.yaml
kubectl apply -f basic-pod.yaml 2>&1
# Expected: Forbidden error

# 3. Compliant pods work
kubectl apply -f restricted-pod.yaml -n restricted-ns
kubectl get pods -n restricted-ns
# Expected: restricted-pod Running

# 4. Cleanup
kubectl delete namespace baseline-ns restricted-ns warn-mode-ns
```

---

## 🧹 Cleanup

```bash
kubectl delete namespace baseline-ns restricted-ns warn-mode-ns
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Pod Security Admission** | Namespace-level pod security enforcement |
| **Baseline** | Prevents known privileged escalations |
| **Restricted** | Heavily restricted pods for security-sensitive workloads |
| **Enforce** | Reject violations |
| **Warn** | Show warnings but allow |
| **Audit** | Log violations but allow |
| **Security Context** | Pod/container-level security settings |

---

## 🚀 What's Next?

Let's add a service mesh:

**[Lab 19: Service Mesh (Istio Basics) →](../19 - Service Mesh Istio/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Pod Security mastered! Your containers are now          ║
║     locked down tighter than Fort Knox! 🏰                 ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
