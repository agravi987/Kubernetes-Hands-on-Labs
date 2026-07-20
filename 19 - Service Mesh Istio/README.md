# Lab 19 — Service Mesh (Istio Basics)

<div align="center">

![Lab-19](https://img.shields.io/badge/Lab-19-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐⭐%20Hard-red)
![Time](https://img.shields.io/badge/Time-45%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Istio-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 19 — Service Mesh (Istio Basics)                        ║
║  "Advanced Traffic Management & Observability"               ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"A service mesh is like adding a GPS, dashcam, and traffic controller to every car in your fleet. Every pod gets a smart proxy that handles traffic, security, and monitoring!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Install Istio on Minikube
- ✅ Enable sidecar injection
- ✅ Deploy sample applications
- ✅ Configure traffic routing (canary deployments)
- ✅ Understand Istio's value proposition

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-18
- [ ] Minikube with at least 4GB RAM
- [ ] istioctl installed

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
⚠️  Istio needs at least 4GB RAM for Minikube!
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    ISTIO SERVICE MESH                            │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Control Plane (istiod)                  │  │
│  │  ├── Pilot: Traffic management                            │  │
│  │  ├── Citadel: Certificate management                     │  │
│  │  └── Galley: Configuration validation                     │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Data Plane (Sidecar Proxies)           │  │
│  │                                                            │  │
│  │  ┌──────────────────┐     ┌──────────────────┐           │  │
│  │  │   Frontend Pod    │     │   Backend Pod     │           │  │
│  │  │  ┌────┐ ┌──────┐ │     │  ┌────┐ ┌──────┐ │           │  │
│  │  │  │App │ │Envoy │◀┼─────┼─▶│Envoy│ │ App  │ │           │  │
│  │  │  └────┘ └──────┘ │     │  └────┘ └──────┘ │           │  │
│  │  │  Sidecar (Envoy)  │     │  Sidecar (Envoy)  │           │  │
│  │  └──────────────────┘     └──────────────────┘           │  │
│  │                                                            │  │
│  │  Traffic flows through Envoy proxies, not directly!       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  CANARY DEPLOYMENT:                                             │
│  ┌──────────┐ 90% traffic ┌──────────────┐                    │
│  │ Frontend │────────────▶│ v1 (stable)  │                    │
│  │  (Envoy) │ 10% traffic │              │                    │
│  │          │────────────▶│ v2 (canary)  │                    │
│  └──────────┘             └──────────────┘                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Install istioctl

```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH

# Verify installation
istioctl version
```

> 💡 **Rithu's Tip:** *"istioctl is Istio's CLI tool, like kubectl is for Kubernetes. You'll use it to install, configure, and troubleshoot Istio!"*

---

### Step 2: Install Istio

```bash
# Install Istio with demo profile (good for learning)
istioctl install --set profile=demo -y
```

Wait for installation to complete. This takes a few minutes.

```bash
# Verify Istio components
kubectl get pods -n istio-system
```

Expected output:
```
NAME                                    READY   STATUS    RESTARTS   AGE
istiod-xxxxxxxxx-xxxxx                 1/1     Running   0          2m
istio-ingressgateway-xxxxxxxxx-xxxxx   1/1     Running   0          2m
```

📸 **Screenshot Placeholder:** *[Terminal showing Istio components running]*

---

### Step 3: Enable Sidecar Injection

```bash
# Enable automatic sidecar injection for the default namespace
kubectl label namespace default istio-injection=enabled

# Verify
kubectl get namespace -L istio-injection
```

Expected output:
```
NAME              STATUS   AGE   ISTIO-INJECTION
default           Active   30m   enabled
kube-node-lease   Active   30m   
kube-public       Active   30m   
kube-system       Active   30m   
```

> 💡 **Rithu's Tip:** *"When you enable sidecar injection, Istio automatically adds an Envoy proxy container to every new pod. It's like giving every pod a smart traffic assistant!"*

---

### Step 4: Deploy Sample Application

```bash
cat > bookinfo.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productpage
  labels:
    app: productpage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: productpage
  template:
    metadata:
      labels:
        app: productpage
    spec:
      containers:
      - name: productpage
        image: istio/examples-bookinfo-productpage-v1:1.20.2
        ports:
        - containerPort: 9080
---
apiVersion: v1
kind: Service
metadata:
  name: productpage
spec:
  selector:
    app: productpage
  ports:
  - port: 9080
    targetPort: 9080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews
  labels:
    app: reviews
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reviews
  template:
    metadata:
      labels:
        app: reviews
        version: v3
    spec:
      containers:
      - name: reviews
        image: istio/examples-bookinfo-reviews-v3:1.20.2
        ports:
        - containerPort: 9080
EOF
```

```bash
kubectl apply -f bookinfo.yaml
kubectl get pods
```

Wait for pods to be Running. Each pod should have 2/2 containers (app + envoy sidecar)!

```bash
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{range .spec.containers[*]}{.name}{" "}{end}{"\n"}{end}'
```

Expected output:
```
productpage-xxxxx    productpage ist-proxy
reviews-xxxxx        reviews ist-proxy
reviews-yyyyy        reviews ist-proxy
```

> 💡 **Rithu's Tip:** *"See 'ist-proxy' in every pod? That's the Envoy sidecar! It intercepts all network traffic, enabling Istio's features."*

📸 **Screenshot Placeholder:** *[Terminal showing pods with sidecar containers]*

---

### Step 5: Traffic Routing — Canary Deployment

```bash
cat > canary-virtual-service.yaml << 'EOF'
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: stable
      weight: 90
    - destination:
        host: reviews
        subset: canary
      weight: 10
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: stable
    labels:
      version: v3
  - name: canary
    labels:
      version: v3
EOF
```

```bash
kubectl apply -f canary-virtual-service.yaml
kubectl get virtualservices
kubectl get destinationrules
```

> 💡 **Rithu's Tip:** *"VirtualService defines WHERE traffic goes. DestinationRule defines WHICH pods handle it. Together, they enable canary deployments, traffic splitting, and more!"*

📸 **Screenshot Placeholder:** *[Terminal showing VirtualService and DestinationRule]*

---

### Step 6: View Istio Dashboard (Kiali)

```bash
# Install Kiali (Istio's dashboard)
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml
```

```bash
# Access Kiali dashboard
istioctl dashboard kiali
```

This opens the Kiali web UI showing your service mesh topology!

📸 **Screenshot Placeholder:** *[Browser showing Kiali dashboard with service mesh topology]*

> 💡 **Rithu's Tip:** *"Kiali gives you a beautiful visualization of your service mesh. You can see traffic flow, health metrics, and configuration all in one place!"*

---

### Step 7: Clean Up

```bash
kubectl delete -f bookinfo.yaml
kubectl delete -f canary-virtual-service.yaml
kubectl delete -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml 2>/dev/null

# Uninstall Istio
istioctl uninstall --purge -y
kubectl delete namespace istio-system

rm -rf istio-*
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. Istio installed
istioctl version
kubectl get pods -n istio-system

# 2. Sidecar injection enabled
kubectl get namespace -L istio-injection
# Expected: default shows "enabled"

# 3. Pods have sidecars
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].name}' | grep -o ist-proxy

# 4. Cleanup
istioctl uninstall --purge -y
kubectl delete namespace istio-system
```

---

## 🧹 Cleanup

```bash
istioctl uninstall --purge -y 2>/dev/null
kubectl delete namespace istio-system 2>/dev/null
kubectl label namespace default istio-injection- 2>/dev/null
kubectl delete -f . 2>/dev/null
rm -rf istio-* *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Service Mesh** | Infrastructure layer for service-to-service communication |
| **Istio** | Most popular service mesh implementation |
| **Sidecar Proxy** | Envoy proxy injected into every pod |
| **VirtualService** | Traffic routing rules |
| **DestinationRule** | Defines subsets and load balancing |
| **Canary Deployment** | Gradually shift traffic to new versions |
| **Kiali** | Service mesh visualization dashboard |

---

## 🚀 What's Next?

You've completed Section 4! Time for production tools:

**[Lab 20: Helm Charts →](../20 - Helm Charts/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Section 4 Complete! Your cluster now has:                ║
║     RBAC, Network Policies, Pod Security, and Istio! 🛡️     ║
║     Your security game is STRONG!                            ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
