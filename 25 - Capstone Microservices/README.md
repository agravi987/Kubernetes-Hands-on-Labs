# Lab 25 — Capstone: Microservices on Kubernetes

<div align="center">

![Lab-25](https://img.shields.io/badge/Lab-25-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐⭐⭐%20Expert-critical)
![Time](https://img.shields.io/badge/Time-90%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20%28Minikube%29-brightgreen)
![Service](https://img.shields.io/badge/Service-Capstone%20Project-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 25 — Capstone: Microservices on Kubernetes              ║
║  "The Ultimate Test: Build, Deploy, Secure, Monitor"         ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"This is it, Ravi — the final boss! Everything you've learned in Labs 01-24 comes together here. A real-world microservices application with frontend, API, database, Ingress, autoscaling, and monitoring. Ready?"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Deploy a multi-service application (frontend + API + database)
- ✅ Configure Ingress routing
- ✅ Set up HPA for autoscaling
- ✅ Add monitoring and logging
- ✅ Package everything with Helm
- ✅ Deploy via rolling updates

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-24
- [ ] Minikube running with 4GB+ RAM
- [ ] Helm installed
- [ ] kubectl configured

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
```

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    CAPSTONE APPLICATION ARCHITECTURE                  │
│                                                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    INGRESS CONTROLLER                          │   │
│  │  ├── /         → frontend-service:80                         │   │
│  │  ├── /api/*    → api-service:3000                            │   │
│  │  └── /health   → api-service:3000/health                     │   │
│  └──────────┬──────────────────────────────────────┬─────────────┘   │
│             │                                      │                  │
│  ┌──────────▼──────────────┐    ┌─────────────────▼───────────┐    │
│  │     Frontend Service     │    │        API Service           │    │
│  │     (Deployment)         │    │        (Deployment)          │    │
│  │     Replicas: 2-5 (HPA) │    │        Replicas: 2-10 (HPA) │    │
│  │                          │    │                               │    │
│  │  ┌──────────────────┐   │    │  ┌──────────────────────┐   │    │
│  │  │   React App       │   │    │  │   Node.js/Express    │   │    │
│  │  │   (nginx:1.25)   │   │    │  │   (node:18-alpine)  │   │    │
│  │  └──────────────────┘   │    │  └──────────┬───────────┘   │    │
│  └──────────────────────────┘    │             │                │    │
│                                  └─────────────┼────────────────┘    │
│                                                │                      │
│                                  ┌─────────────▼────────────────┐    │
│                                  │        PostgreSQL             │    │
│                                  │        (StatefulSet)          │    │
│                                  │        Replicas: 1            │    │
│                                  │        PVC: 1Gi               │    │
│                                  │        Secret: credentials    │    │
│                                  └──────────────────────────────┘    │
│                                                                       │
│  MONITORING:                                                          │
│  ├── Prometheus (scrapes /metrics from API)                         │
│  ├── Grafana (dashboards)                                            │
│  └── PrometheusRule (alerts)                                         │
│                                                                       │
│  CONFIGURATION:                                                       │
│  ├── ConfigMap: APP_CONFIG                                           │
│  ├── Secret: DB_CREDENTIALS                                         │
│  └── ConfigMap: NGINX_CONFIG                                        │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create Project Structure

```bash
mkdir -p capstone/{frontend,api,helm-chart/templates}
cd capstone
```

---

### Step 2: Create the Database (PostgreSQL StatefulSet)

```bash
cat > helm-chart/templates/postgres.yaml << 'EOF'
apiVersion: v1
kind: Secret
metadata:
  name: postgres-credentials
data:
  POSTGRES_USER: YWRtaW4=          # admin
  POSTGRES_PASSWORD: cEBzc3cwcmQ=  # p@ssw0rd
  POSTGRES_DB: Y2Fwc3RvbmVfZGI=     # capstone_db
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
        envFrom:
        - secretRef:
            name: postgres-credentials
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            cpu: "250m"
            memory: "512Mi"
          limits:
            cpu: "500m"
            memory: "1Gi"
        readinessProbe:
          exec:
            command: ["pg_isready", "-U", "admin", "-d", "capstone_db"]
          initialDelaySeconds: 5
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
EOF
```

---

### Step 3: Create the API Service

```bash
cat > helm-chart/templates/api.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
data:
  NODE_ENV: "production"
  PORT: "3000"
  DATABASE_HOST: "postgres-service"
  DATABASE_PORT: "5432"
  DATABASE_NAME: "capstone_db"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  labels:
    app: api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: node:18-alpine
        command: ["sh", "-c"]
        args:
        - |
          cat > /app/server.js << 'SERVEREOF'
          const http = require('http');
          const hostname = '0.0.0.0';
          const port = process.env.PORT || 3000;

          let requestCount = 0;

          const server = http.createServer((req, res) => {
            requestCount++;
            res.writeHead(200, { 'Content-Type': 'application/json' });
            
            if (req.url === '/health') {
              res.end(JSON.stringify({ status: 'healthy', timestamp: new Date().toISOString() }));
            } else if (req.url === '/metrics') {
              res.end(JSON.stringify({ requests: requestCount, uptime: process.uptime() }));
            } else {
              res.end(JSON.stringify({
                message: 'Welcome to the Capstone API!',
                version: '1.0.0',
                endpoints: ['/health', '/metrics', '/api/data'],
                requestId: requestCount
              }));
            }
          });

          server.listen(port, hostname, () => {
            console.log(`API server running on port ${port}`);
          });
          SERVEREOF
          node /app/server.js
        ports:
        - containerPort: 3000
        envFrom:
        - configMapRef:
            name: api-config
        - secretRef:
            name: postgres-credentials
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 15
          periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api
  ports:
  - port: 3000
    targetPort: 3000
EOF
```

---

### Step 4: Create the Frontend

```bash
cat > helm-chart/templates/frontend.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  default.conf: |
    server {
        listen 80;
        server_name localhost;

        location / {
            root /usr/share/nginx/html;
            index index.html;
            try_files $uri $uri/ /index.html;
        }

        location /api/ {
            proxy_pass http://api-service:3000;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        location /health {
            return 200 'OK';
            add_header Content-Type text/plain;
        }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
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
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d/
        - name: html
          mountPath: /usr/share/nginx/html/
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "250m"
            memory: "128Mi"
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
      initContainers:
      - name: create-index
        image: busybox:latest
        command: ['sh', '-c']
        args:
        - |
          cat > /html/index.html << 'HTMLEOF'
          <!DOCTYPE html>
          <html>
          <head>
            <title>Capstone App</title>
            <style>
              body { font-family: Arial, sans-serif; text-align: center; padding: 50px; background: #1a1a2e; color: #e94560; }
              h1 { font-size: 3em; }
              .card { background: #16213e; padding: 20px; margin: 20px auto; border-radius: 10px; max-width: 500px; }
              .status { color: #0f3460; background: #4ecca3; padding: 10px; border-radius: 5px; display: inline-block; }
            </style>
          </head>
          <body>
            <h1>Kubernetes Capstone</h1>
            <div class="card">
              <h2>Frontend + API + Database</h2>
              <p>This microservices app is running on Kubernetes!</p>
              <p class="status">All Systems Operational</p>
            </div>
            <div class="card">
              <h3>API Response:</h3>
              <pre id="api-response">Loading...</pre>
            </div>
            <script>
              fetch('/api/').then(r => r.json()).then(d => {
                document.getElementById('api-response').textContent = JSON.stringify(d, null, 2);
              }).catch(() => {
                document.getElementById('api-response').textContent = 'API not available yet';
              });
            </script>
          </body>
          </html>
          HTMLEOF
        volumeMounts:
        - name: html
          mountPath: /html
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config
      - name: html
        emptyDir: {}
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
EOF
```

---

### Step 5: Create HPA for Autoscaling

```bash
cat > helm-chart/templates/hpa.yaml << 'EOF'
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
EOF
```

---

### Step 6: Create Ingress

```bash
cat > helm-chart/templates/ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: capstone-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
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
            name: api-service
            port:
              number: 3000
EOF
```

---

### Step 7: Deploy Everything!

```bash
# Make sure Ingress addon is enabled
minikube addons enable ingress

# Deploy the entire application
kubectl apply -f helm-chart/templates/

# Watch pods come up
kubectl get pods -w
```

Wait for all pods to be Running:
```
NAME                        READY   STATUS    RESTARTS   AGE
api-xxxxx-xxxxx             1/1     Running   0          1m
api-yyyyy-yyyyy             1/1     Running   0          1m
frontend-xxxxx-xxxxx        1/1     Running   0          1m
frontend-yyyyy-yyyyy        1/1     Running   0          1m
postgres-0                  1/1     Running   0          2m
```

📸 **Screenshot Placeholder:** *[Terminal showing all capstone pods running]*

---

### Step 8: Test the Application

```bash
# Get the Ingress URL
INGRESS_URL=$(minikube service ingress-nginx-controller -n ingress-nginx --url | head -1)
echo "App URL: $INGRESS_URL"
```

```bash
# Test frontend
curl $INGRESS_URL/
# Expected: HTML page

# Test API
curl $INGRESS_URL/api/
# Expected: {"message":"Welcome to the Capstone API!",...}

# Test health
curl $INGRESS_URL/health
# Expected: OK

# Test metrics
curl $INGRESS_URL/api/metrics
# Expected: {"requests":N,"uptime":X}
```

📸 **Screenshot Placeholder:** *[Browser showing capstone app]*

> 💡 **Rithu's Tip:** *"If you see the frontend page and can hit the API, you've built a working microservices architecture on Kubernetes!"*

---

### Step 9: Verify All Components

```bash
echo "=== PODS ==="
kubectl get pods

echo ""
echo "=== SERVICES ==="
kubectl get svc

echo ""
echo "=== INGRESS ==="
kubectl get ingress

echo ""
echo "=== HPA ==="
kubectl get hpa

echo ""
echo "=== SECRETS ==="
kubectl get secrets

echo ""
echo "=== CONFIGMAPS ==="
kubectl get configmaps

echo ""
echo "=== PVC ==="
kubectl get pvc
```

📸 **Screenshot Placeholder:** *[Terminal showing all components]*

---

### Step 10: Scale Test

```bash
# Generate load to trigger autoscaling
kubectl run load-test --image=busybox:latest --rm -it --restart=Never -- sh -c \
  "while true; do wget -qO- http://api-service:3000/; done" &

# Watch HPA
kubectl get hpa -w
```

Wait for the API HPA to show scaling activity.

---

### Step 11: Clean Up

```bash
# Delete all capstone resources
kubectl delete -f helm-chart/templates/

# Verify everything is gone
kubectl get all
```

```bash
# Remove project files
cd ..
rm -rf capstone
```

---

## ✅ Verification

```bash
# 1. All pods running
kubectl get pods | grep -E "api|frontend|postgres"
# Expected: All pods Running

# 2. Services created
kubectl get svc | grep -E "api|frontend|postgres"
# Expected: 3 services

# 3. Ingress works
kubectl get ingress capstone-ingress
# Expected: ADDRESS assigned

# 4. HPA exists
kubectl get hpa
# Expected: frontend-hpa and api-hpa

# 5. API responds
curl http://INGRESS_URL/api/
# Expected: JSON response

# 6. Cleanup
kubectl delete -f helm-chart/templates/
```

---

## 🧹 Cleanup

```bash
kubectl delete -f helm-chart/templates/ 2>/dev/null
kubectl delete ingress --all 2>/dev/null
kubectl delete hpa --all 2>/dev/null
kubectl delete svc --all 2>/dev/null
kubectl delete deploy --all 2>/dev/null
kubectl delete statefulset --all 2>/dev/null
kubectl delete pvc --all 2>/dev/null
kubectl delete secret --all 2>/dev/null
kubectl delete configmap --all 2>/dev/null
cd .. && rm -rf capstone
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Multi-Service Architecture** | Frontend + API + Database in Kubernetes |
| **StatefulSet** | Running databases with persistent storage |
| **HPA** | Auto-scaling based on CPU metrics |
| **Ingress** | Routing external traffic to services |
| **ConfigMaps** | Externalizing configuration |
| **Secrets** | Managing sensitive data |
| **Readiness/Liveness Probes** | Health checking |
| **Init Containers** | Setup tasks before main container starts |

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉🎉🎉 CONGRATULATIONS, RAVI! 🎉🎉🎉                    ║
║                                                              ║
║  You've completed ALL 25 Kubernetes Hands-On Labs!           ║
║                                                              ║
║  From "What's a pod?" to deploying a full                    ║
║  microservices application — you've come so far!             ║
║                                                              ║
║  You now know:                                               ║
║  ├── Pods, ReplicaSets, Deployments, Services               ║
║  ├── ConfigMaps, Secrets, PV/PVC, StatefulSets              ║
║  ├── Ingress, Namespaces, Resource Quotas                   ║
║  ├── Jobs, CronJobs, DaemonSets, HPA, PDBs                  ║
║  ├── RBAC, Network Policies, Pod Security                   ║
║  ├── Service Mesh (Istio), Helm Charts                      ║
║  ├── Monitoring (Prometheus/Grafana), Logging (EFK)          ║
║  ├── Backup (Velero), Cloud (EKS)                           ║
║  └── Full Microservices Architecture                        ║
║                                                              ║
║  "The best way to predict the future is to create it."     ║
║                                                    — Rithu  ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
