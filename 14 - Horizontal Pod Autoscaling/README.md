# Lab 14 — Horizontal Pod Autoscaling

<div align="center">

![Lab-14](https://img.shields.io/badge/Lab-14-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Med--Hard-orange)
![Time](https://img.shields.io/badge/Time-35%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-HPA-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 14 — Horizontal Pod Autoscaling                         ║
║  "Scale Automatically Based on Demand"                       ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"HPA is like a smart elevator operator — when there are too many people, more elevators show up. When it's quiet, they disappear. No wasted resources!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Create an HPA based on CPU utilization
- ✅ Create an HPA based on memory utilization
- ✅ Load test your app and watch scaling
- ✅ Understand min/max replica configuration

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-13
- [ ] Minikube running
- [ ] Metrics Server enabled

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    HPA SCALING FLOW                              │
│                                                                  │
│  ┌──────────────────┐                                           │
│  │  Metrics Server   │  ← Collects CPU/Memory metrics           │
│  └────────┬─────────┘                                           │
│           │                                                      │
│           ▼                                                      │
│  ┌──────────────────┐                                           │
│  │  HPA Controller   │  ← Compares current vs target metrics   │
│  └────────┬─────────┘                                           │
│           │                                                      │
│           ▼                                                      │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Deployment / ReplicaSet                               │      │
│  │                                                        │      │
│  │  Current CPU: 80%  Target CPU: 50%  → Scale UP! 📈   │      │
│  │  Current CPU: 20%  Target CPU: 50%  → Scale DOWN! 📉 │      │
│  │                                                        │      │
│  │  Min Replicas: 2   Max Replicas: 10                   │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Enable Metrics Server

```bash
# Enable metrics server addon in Minikube
minikube addons enable metrics-server
```

```bash
# Verify metrics server is running
kubectl get pods -n kube-system | grep metrics-server
```

Expected output:
```
metrics-server-xxxxxxxxx-xxxxx   1/1     Running   0          1m
```

```bash
# Wait for metrics to be available
kubectl top nodes
```

> 💡 **Rithu's Tip:** *"The Metrics Server collects resource usage data from every node and pod. Without it, HPA can't work!"*

📸 **Screenshot Placeholder:** *[Terminal showing metrics server running]*

---

### Step 2: Create a Deployment with Resource Requests

```bash
cat > php-apache-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "200m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
spec:
  type: NodePort
  selector:
    app: php-apache
  ports:
  - port: 80
    targetPort: 80
EOF
```

```bash
kubectl apply -f php-apache-deployment.yaml
kubectl get pods -l app=php-apache
```

Wait for the pod to be Running.

---

### Step 3: Create CPU-based HPA

```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

```bash
# Check the HPA
kubectl get hpa
```

Expected output:
```
NAME         REFERENCE                     TARGETS         MINPODS   MAXPODS   REPLICAS
php-apache   Deployment/php-apache/scale   0%/50%          1         10        1
```

> 💡 **Rithu's Tip:** *"TARGETS shows 0%/50% — that means current CPU is 0% and we're targeting 50%. When CPU exceeds 50%, more pods will be created!"*

📸 **Screenshot Placeholder:** *[Terminal showing HPA with 0% target]*

---

### Step 4: Load Test the Application

```bash
# Start a load generator in a separate terminal
kubectl run load-generator --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside the load generator pod:

```bash
# Create a tight loop to stress the CPU
while true; do wget -q -O- http://php-apache; done
```

**In another terminal**, watch the HPA:

```bash
kubectl get hpa -w
```

After 1-2 minutes, you should see:

```
NAME         REFERENCE                     TARGETS         MINPODS   MAXPODS   REPLICAS
php-apache   Deployment/php-apache/scale   52%/50%         1         10        1
php-apache   Deployment/php-apache/scale   75%/50%         1         10        3
php-apache   Deployment/php-apache/scale   90%/50%         1         10        5
php-apache   Deployment/php-apache/scale   45%/50%         1         10        5
php-apache   Deployment/php-apache/scale   25%/50%         1         10        3
php-apache   Deployment/php-apache/scale   5%/50%          1         10        2
```

📸 **Screenshot Placeholder:** *[Terminal showing HPA scaling up and down]*

> 💡 **Rithu's Tip:** *"When load hits 50% CPU, HPA creates more pods. When load decreases, it scales back down. The delay is usually 1-2 minutes — HPA needs time to collect metrics!"*

---

### Step 5: Create HPA using YAML (Advanced)

```bash
cat > hpa.yaml << 'EOF'
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Pods
        value: 2
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
EOF
```

```bash
# Delete the kubectl-created HPA first
kubectl delete hpa php-apache

# Apply the YAML-based HPA
kubectl apply -f hpa.yaml
kubectl get hpa
```

> 💡 **Rithu's Tip:** *"The behavior section controls scaling speed! scaleUp is fast (add 2 pods every 60s), scaleDown is slow (reduce by 10% every 60s). This prevents flapping!"*

📸 **Screenshot Placeholder:** *[Terminal showing advanced HPA configuration]*

---

### Step 6: View Current Metrics

```bash
# Check pod resource usage
kubectl top pods -l app=php-apache
```

Expected output:
```
NAME                         CPU(cores)   MEMORY(bytes)
php-apache-xxxxxxxxx-xxxxx   5m           8Mi
```

```bash
# Check node resource usage
kubectl top nodes
```

Expected output:
```
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
minikube   200m         10%    500Mi           25%
```

> 💡 **Rithu's Tip:** *"kubectl top gives you real-time resource usage. It's like 'top' or 'htop' but for Kubernetes!"*

---

### Step 7: Stop Load and Watch Scale Down

Press `Ctrl+C` in the load generator pod.

Wait 3-5 minutes and watch:

```bash
kubectl get hpa -w
```

The replicas will gradually decrease back to 1.

---

### Step 8: Clean Up

```bash
# Delete load generator (if still running)
kubectl delete pod load-generator 2>/dev/null

kubectl delete hpa --all
kubectl delete -f php-apache-deployment.yaml
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. Metrics server running
kubectl top nodes
# Expected: CPU and memory metrics shown

# 2. HPA created
kubectl get hpa
# Expected: php-apache with TARGETS showing percentage

# 3. Scaling works
# (Run load test and observe HPA)
kubectl get hpa -w
# Expected: TARGETS increase, REPLICAS increase

# 4. Cleanup
kubectl delete hpa --all
kubectl delete deploy --all
```

---

## 🧹 Cleanup

```bash
kubectl delete hpa --all
kubectl delete deploy --all
kubectl delete svc --all
kubectl delete pod --all
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **HPA** | Automatically scales pods based on metrics |
| **CPU Utilization** | Scale based on CPU usage percentage |
| **Memory Utilization** | Scale based on memory usage percentage |
| **Metrics Server** | Provides resource usage data to HPA |
| **Scale Behavior** | Control how fast HPA scales up/down |
| **Min/Max Replicas** | Set boundaries for scaling |

---

## 🚀 What's Next?

Let's protect apps during maintenance:

**[Lab 15: Pod Disruption Budgets →](../15 - Pod Disruption Budgets/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Your apps now scale automatically!                      ║
║     Traffic spike? No problem! Traffic drop? No waste! 📈📉 ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
