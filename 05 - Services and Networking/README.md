# Lab 05 — Services & Networking

<div align="center">

![Lab-05](https://img.shields.io/badge/Lab-05-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐%20Easy--Med-blue)
![Time](https://img.shields.io/badge/Time-35%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Services%20%2F%20Networking-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 05 — Services & Networking                              ║
║  "How Pods Talk to Each Other (and the World)"               ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Pods are like people at a party — they come and go. Services are like the party host — they always know where everyone is and can direct guests to the right room!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand why Services exist
- ✅ Create ClusterIP, NodePort, and LoadBalancer services
- ✅ Use DNS-based service discovery
- ✅ Expose applications externally
- ✅ Understand kube-proxy and iptables rules

---

## 🧠 Prerequisites

- [ ] Completed Lab 04 (Deployments)
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
│                      KUBERNETES CLUSTER                          │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                    CLUSTER-INTERNAL                     │     │
│  │                                                         │     │
│  │  ┌─────────────────────────────────────────────────┐   │     │
│  │  │           ClusterIP Service                     │   │     │
│  │  │           (Internal only - 10.96.x.x)           │   │     │
│  │  │                                                   │   │     │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐         │   │     │
│  │  │  │  Pod 1  │←→│  Pod 2  │←→│  Pod 3  │         │   │     │
│  │  │  │ (nginx) │  │ (nginx) │  │ (nginx) │         │   │     │
│  │  │  └─────────┘  └─────────┘  └─────────┘         │   │     │
│  │  └─────────────────────────────────────────────────┘   │     │
│  │                                                         │     │
│  │  ┌─────────────────────────────────────────────────┐   │     │
│  │  │           NodePort Service                       │   │     │
│  │  │           (Node access - port 30080)             │   │     │
│  │  │                                                   │   │     │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐         │   │     │
│  │  │  │  Pod 1  │←→│  Pod 2  │←→│  Pod 3  │         │   │     │
│  │  │  └─────────┘  └─────────┘  └─────────┘         │   │     │
│  │  └──────────────────────┬──────────────────────────┘   │     │
│  │                         │                               │     │
│  └─────────────────────────┼──────────────────────────────┘     │
│                            │                                     │
│  ┌─────────────────────────┼──────────────────────────────┐     │
│  │           Node (minikube)│                              │     │
│  │                          ▼                              │     │
│  │  ┌─────────────────────────────────────────────────┐   │     │
│  │  │  localhost:30080 → Service → Pod                 │   │     │
│  │  └─────────────────────────────────────────────────┘   │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  ┌──────────────────────────────────────────────────────┐      │
│  │           LoadBalancer Service                        │      │
│  │           (External - minikube tunnel)                │      │
│  │           (Cloud: provisions cloud LB)                │      │
│  └──────────────────────────────────────────────────────┘      │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create a Deployment to Expose

First, let's create something to expose:

```bash
cat > nginx-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
EOF
```

```bash
kubectl apply -f nginx-deployment.yaml
kubectl get pods -l app=nginx
```

Wait until all 3 pods are Running, then proceed.

---

### Step 2: Create a ClusterIP Service

ClusterIP is the **default** service type — internal-only access.

```bash
cat > clusterip-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
EOF
```

```bash
kubectl apply -f clusterip-service.yaml
kubectl get services
```

Expected output:
```
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP   10m
nginx-clusterip    ClusterIP   10.108.131.42   <none>        80/TCP    10s
```

```bash
# Check the endpoints (which pods this service points to)
kubectl get endpoints nginx-clusterip
```

Expected output:
```
NAME               ENDPOINTS                                      AGE
nginx-clusterip    172.17.0.2:80,172.17.0.3:80,172.17.0.4:80    30s
```

> 💡 **Rithu's Tip:** *"ClusterIP services are only accessible from within the cluster. Perfect for microservices talking to each other, but useless for external users."*

📸 **Screenshot Placeholder:** *[Terminal showing ClusterIP service and endpoints]*

---

### Step 3: Test ClusterIP with DNS

```bash
# Start a test pod to access the ClusterIP service
kubectl run test-pod --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside the test pod, try these commands:

```bash
# Method 1: By service name (DNS discovery)
wget -qO- http://nginx-clusterip

# Method 2: By ClusterIP
wget -qO- http://10.108.131.42

# Method 3: Full DNS name
wget -qO- http://nginx-clusterip.default.svc.cluster.local

exit
```

> 💡 **Rithu's Tip:** *"Kubernetes DNS automatically creates DNS entries for services. Service name = DNS name. No need to remember IPs!"*

📸 **Screenshot Placeholder:** *[Terminal showing successful DNS resolution inside test pod]*

---

### Step 4: Create a NodePort Service

NodePort exposes the service on each node's IP at a static port (30000-32767).

```bash
cat > nodeport-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    protocol: TCP
EOF
```

```bash
kubectl apply -f nodeport-service.yaml
kubectl get services
```

Expected output:
```
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP        15m
nginx-clusterip    ClusterIP   10.108.131.42   <none>        80/TCP         5m
nginx-nodeport     NodePort    10.100.50.100   <none>        80:30080/TCP   10s
```

```bash
# For Minikube, use this command to get the URL
minikube service nginx-nodeport --url
```

Open the URL in your browser — you should see the nginx welcome page!

📸 **Screenshot Placeholder:** *[Browser showing nginx welcome page via NodePort]*

> 💡 **Rithu's Tip:** *"NodePort is great for development but not ideal for production. You'd need to know the node's IP, and the port range is limited to 30000-32767. LoadBalancer or Ingress is better for prod!"*

---

### Step 5: Create a LoadBalancer Service

On cloud providers, LoadBalancer provisions an external load balancer. On Minikube, we use `minikube tunnel`:

```bash
# Open a new terminal and run:
minikube tunnel
# Enter your password when prompted
# Keep this running in the background!
```

```bash
cat > loadbalancer-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
EOF
```

```bash
kubectl apply -f loadbalancer-service.yaml
```

```bash
# Wait a moment, then check
kubectl get services nginx-loadbalancer
```

Expected output:
```
NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
nginx-loadbalancer   LoadBalancer   10.100.200.50   10.100.200.50   80:3XXXX/TCP   30s
```

Open **http://10.100.200.50** in your browser — nginx welcome page!

> 💡 **Rithu's Tip:** *"On Minikube, LoadBalancer only works with minikube tunnel running. On AWS/GCP/Azure, it provisions a real load balancer (like an ELB). That's when costs start!"*

📸 **Screenshot Placeholder:** *[Terminal showing LoadBalancer service with external IP]*

---

### Step 6: Service Discovery Deep Dive

Let's explore how services find each other:

```bash
# Start an interactive pod
kubectl run discovery-pod --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside the pod:

```bash
# DNS lookup for the service
nslookup nginx-clusterip

# DNS lookup with full FQDN
nslookup nginx-clusterip.default.svc.cluster.local

# Get environment variables (services auto-injected as env vars)
env | grep NGINX

# You'll see something like:
# NGINX_CLUSTERIP_SERVICE_HOST=10.108.131.42
# NGINX_CLUSTERIP_SERVICE_PORT=80
# NGINX_NODEPORT_SERVICE_HOST=10.100.50.100
# NGINX_NODEPORT_SERVICE_PORT=80

exit
```

> 💡 **Rithu's Tip:** *"Kubernetes automatically injects service connection info as environment variables! Each service creates env vars like SERVICE_HOST and SERVICE_PORT for every active service in the namespace."*

---

### Step 7: Multi-Port Services

Real-world services often expose multiple ports:

```bash
cat > multiport-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: https
    port: 443
    targetPort: 443
EOF
```

```bash
kubectl apply -f multiport-service.yaml
kubectl describe service multi-port-service
```

> 💡 **Rithu's Tip:** *"When a service has multiple ports, you MUST name them. This helps with DNS and makes your service definitions clearer!"*

---

### Step 8: Understanding Service Selectors

```bash
# Create a service with a specific selector
cat > frontend-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: ClusterIP
  selector:
    app: nginx
    tier: frontend
  ports:
  - port: 80
    targetPort: 80
EOF
```

```bash
kubectl apply -f frontend-service.yaml

# Check endpoints - should only match pods with BOTH labels
kubectl get endpoints frontend-service
```

Now create a pod WITHOUT the tier label:

```bash
cat > wrong-label-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: wrong-pod
  labels:
    app: nginx
    tier: backend
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
EOF
```

```bash
kubectl apply -f wrong-label-pod.yaml

# Check endpoints again - wrong-pod won't be included!
kubectl get endpoints frontend-service
```

> 💡 **Rithu's Tip:** *"Service selectors are like a strict bouncer — pods MUST match ALL selector labels to be included. No partial matches allowed!"*

---

### Step 9: Headless Services

Sometimes you want direct pod IPs instead of a single service IP:

```bash
cat > headless-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: headless-nginx
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
EOF
```

```bash
kubectl apply -f headless-service.yaml

# DNS lookup returns individual pod IPs!
kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- nslookup headless-nginx
```

> 💡 **Rithu's Tip:** *"Headless services are essential for StatefulSets (like databases) where you need stable network identities for each pod!"*

---

### Step 10: Clean Up

```bash
# Delete all services and deployments
kubectl delete -f .
kubectl delete pod wrong-pod 2>/dev/null
kubectl delete pod test-pod 2>/dev/null
kubectl delete pod discovery-pod 2>/dev/null

# Verify clean slate
kubectl get all
```

```bash
# Stop minikube tunnel if running (Ctrl+C in the tunnel terminal)
```

```bash
rm *.yaml
```

---

## ✅ Verification

```bash
# 1. ClusterIP service works
kubectl apply -f clusterip-service.yaml
kubectl get svc nginx-clusterip
# Expected: CLUSTER-IP assigned

# 2. Endpoints are populated
kubectl get endpoints nginx-clusterip
# Expected: 3 pod IPs listed

# 3. NodePort works
kubectl apply -f nodeport-service.yaml
minikube service nginx-nodeport --url
# Expected: URL to access nginx

# 4. DNS works
kubectl run test --image=busybox --rm -it --restart=Never -- nslookup nginx-clusterip
# Expected: Name resolution works

# 5. Cleanup
kubectl delete svc --all
kubectl delete deploy --all
```

---

## 🧹 Cleanup

```bash
kubectl delete svc --all
kubectl delete deploy --all
kubectl delete pod --all
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Service** | Stable network endpoint for a set of pods |
| **ClusterIP** | Internal-only service (default type) |
| **NodePort** | Exposes service on node's IP at a static port |
| **LoadBalancer** | Provisions external load balancer |
| **Headless Service** | Returns individual pod IPs instead of a single IP |
| **DNS Discovery** | Services are discoverable via DNS names |
| **Selectors** | Label-based matching for service-to-pod routing |

---

## 🚀 What's Next?

Now that you can deploy and expose apps, let's learn about configuration:

**[Lab 06: ConfigMaps & Secrets →](../06 - ConfigMaps and Secrets/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 You've completed Section 1: Fundamentals!              ║
║     Your Kubernetes foundation is SOLID. 💪                 ║
║                                                              ║
║  Ready for the next level? Let's talk config & storage!     ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
