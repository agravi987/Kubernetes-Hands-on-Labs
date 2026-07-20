# Lab 09 — Ingress Controller

<div align="center">

![Lab-09](https://img.shields.io/badge/Lab-09-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Medium-purple)
![Time](https://img.shields.io/badge/Time-40%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Ingress%20Controller-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 09 — Ingress Controller                                 ║
║  "Your Smart Traffic Director for the Web"                   ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"NodePort is like having one door for every guest. Ingress is like having a smart receptionist who directs guests to the right room based on their URL!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand what an Ingress Controller is
- ✅ Install NGINX Ingress Controller in Minikube
- ✅ Create path-based routing rules
- ✅ Understand TLS termination (conceptual)
- ✅ Configure name-based virtual hosts

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-08
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
│                     INGRESS ARCHITECTURE                          │
│                                                                   │
│  External Traffic                                                │
│        │                                                          │
│        ▼                                                          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              NGINX Ingress Controller                     │   │
│  │              (runs as a pod in the cluster)               │   │
│  │                                                           │   │
│  │  Rules:                                                   │   │
│  │  /app/*        → app-service (port 80)                   │   │
│  │  /api/*        → api-service (port 8080)                 │   │
│  │  /grafana/*    → grafana-service (port 3000)             │   │
│  │  *.example.com → different-service                       │   │
│  └──────────┬────────────────┬────────────────┬─────────────┘   │
│             │                │                │                   │
│  ┌──────────▼──────┐ ┌──────▼──────────┐ ┌───▼─────────────┐  │
│  │   app-service   │ │   api-service   │ │ grafana-service  │  │
│  │   (Deployment)  │ │   (Deployment)  │ │ (Deployment)     │  │
│  │   Port: 80      │ │   Port: 8080    │ │ Port: 3000       │  │
│  │   ┌─────────┐  │ │   ┌─────────┐  │ │ ┌─────────┐      │  │
│  │   │  Pods   │  │ │   │  Pods   │  │ │ │  Pods   │      │  │
│  │   └─────────┘  │ │   └─────────┘  │ │ └─────────┘      │  │
│  └────────────────┘ └────────────────┘ └───────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Enable NGINX Ingress Controller in Minikube

```bash
# Enable the NGINX Ingress addon
minikube addons enable ingress
```

```bash
# Verify it's running
kubectl get pods -n ingress-nginx
```

Expected output:
```
NAME                                        READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-xxxxxxxxx-xxxxx   1/1     Running   0          1m
```

📸 **Screenshot Placeholder:** *[Terminal showing ingress-nginx controller running]*

> 💡 **Rithu's Tip:** *"The Ingress Controller is just a regular pod running in the cluster! It watches for Ingress resources and configures itself accordingly."*

---

### Step 2: Create Backend Applications

Let's create two apps to route traffic to:

```bash
cat > backend-apps.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
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
  name: frontend-service
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
  name: backend-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: api
        image: hashicorp/http-echo
        args:
        - "-text=Hello from the API!"
        - "-listen=:5678"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 5678
EOF
```

```bash
kubectl apply -f backend-apps.yaml
kubectl get pods
```

Wait until all pods are Running.

---

### Step 3: Create a Basic Ingress

```bash
cat > basic-ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: basic-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /frontend
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
EOF
```

```bash
kubectl apply -f basic-ingress.yaml
kubectl get ingress
```

Expected output:
```
NAME              CLASS   HOSTS   ADDRESS          PORTS   AGE
basic-ingress     nginx   *       192.168.49.2     80      10s
```

```bash
# Get the Ingress URL
minikube service ingress-nginx-controller -n ingress-nginx --url
```

📸 **Screenshot Placeholder:** *[Terminal showing Ingress resource]*

---

### Step 4: Test Path-Based Routing

```bash
# Get the ingress URL
INGRESS_URL=$(minikube service ingress-nginx-controller -n ingress-nginx --url | head -1)
echo "Ingress URL: $INGRESS_URL"
```

```bash
# Test frontend route
curl $INGRESS_URL/frontend
# Expected: nginx welcome page (HTML)

# Test API route
curl $INGRESS_URL/api
# Expected: Hello from the API!
```

📸 **Screenshot Placeholder:** *[Terminal showing curl responses]*

> 💡 **Rithu's Tip:** *"The Ingress Controller reads the path and forwards traffic to the correct service. /frontend goes to frontend-service, /api goes to backend-service. Magic!"*

---

### Step 5: Name-Based Virtual Hosts

Let's route traffic based on domain names:

```bash
cat > vhost-ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vhost-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: frontend.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: api.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
EOF
```

```bash
kubectl apply -f vhost-ingress.yaml
```

```bash
# Test with host headers
curl --header "Host: frontend.local" $INGRESS_URL/
# Expected: nginx welcome page

curl --header "Host: api.local" $INGRESS_URL/
# Expected: Hello from the API!
```

> 💡 **Rithu's Tip:** *"In production, you'd set up DNS to point different subdomains (app.example.com, api.example.com) to your Ingress Controller's IP!"*

---

### Step 6: TLS Termination (Conceptual)

```bash
cat > tls-ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - secure.example.com
    secretName: tls-secret
  rules:
  - host: secure.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
EOF
```

```bash
# Create a self-signed TLS secret (for testing only!)
kubectl create secret tls tls-secret \
  --cert=<(openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /dev/stdout -out /dev/stdout 2>/dev/null) \
  --key=/dev/stdin 2>/dev/null <<< "$(openssl req -x509 -nodes -days 365 -newkey rsa:2048 2>/dev/null)"
```

> 💡 **Rithu's Tip:** *"In production, you'd use cert-manager with Let's Encrypt for free TLS certificates. The annotation 'cert-manager.io/cluster-issuer: letsencrypt-prod' makes it automatic!"*

---

### Step 7: Ingress Annotations

NGINX Ingress supports many useful annotations:

```bash
cat > advanced-ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: advanced-ingress
  annotations:
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "10"
    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    # Proxy timeouts
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "10"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "120"
    # Body size
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    # Custom headers
    nginx.ingress.kubernetes.io/configuration-snippet: |
      more_set_headers "X-Frame-Options: DENY";
      more_set_headers "X-Content-Type-Options: nosniff";
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
EOF
```

```bash
kubectl apply -f advanced-ingress.yaml
```

> 💡 **Rithu's Tip:** *"Annotations are like plugin features for your Ingress. Rate limiting, CORS, timeouts — all configurable without touching your application code!"*

📸 **Screenshot Placeholder:** *[Terminal showing advanced Ingress configuration]*

---

### Step 8: View Ingress Controller Logs

```bash
# See how the Ingress Controller processes requests
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --follow
```

> 💡 **Rithu's Tip:** *"The logs show exactly how requests are routed. If something's not working, check the logs first!"*

---

### Step 9: Clean Up

```bash
kubectl delete -f .
kubectl delete ingress --all
kubectl delete secret tls-secret 2>/dev/null
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. Ingress Controller is running
kubectl get pods -n ingress-nginx
# Expected: Controller pod in Running state

# 2. Ingress resource exists
kubectl get ingress basic-ingress
# Expected: ADDRESS assigned

# 3. Path routing works
curl $INGRESS_URL/api
# Expected: Hello from the API!

# 4. Cleanup
kubectl delete ingress --all
```

---

## 🧹 Cleanup

```bash
kubectl delete ingress --all
kubectl delete deploy --all
kubectl delete svc --all
kubectl delete secret tls-secret 2>/dev/null
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Ingress** | Rules for routing external HTTP/HTTPS traffic to services |
| **Ingress Controller** | The actual proxy (NGINX) that processes Ingress rules |
| **Path-based Routing** | Route traffic based on URL path |
| **Name-based Virtual Hosts** | Route traffic based on domain name |
| **TLS Termination** | Handle HTTPS at the Ingress layer |
| **Annotations** | Configure Ingress Controller behavior |
| **Rewrite Target** | Modify request paths before forwarding |

---

## 🚀 What's Next?

Let's learn about organizing and resource-managing your cluster:

**[Lab 10: Namespaces & Resource Quotas →](../10 - Namespaces and Resource Quotas/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Ingress mastered! Your cluster now has a smart          ║
║     front door for the internet! 🚪                         ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
