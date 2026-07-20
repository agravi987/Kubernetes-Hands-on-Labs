# Lab 06 — ConfigMaps & Secrets

<div align="center">

![Lab-06](https://img.shields.io/badge/Lab-06-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐%20Easy--Med-blue)
![Time](https://img.shields.io/badge/Time-30%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-ConfigMaps%20%2F%20Secrets-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 06 — ConfigMaps & Secrets                               ║
║  "Keep Your Config Clean and Your Secrets Safe"              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Hardcoding configuration in your Docker images is like writing your password on a sticky note on your monitor. Don't do it!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Create ConfigMaps from literals, files, and directories
- ✅ Create Secrets for sensitive data
- ✅ Inject configuration as environment variables
- ✅ Mount ConfigMaps and Secrets as volumes
- ✅ Understand base64 encoding

---

## 🧠 Prerequisites

- [ ] Completed Section 1 (Labs 01-05)
- [ ] Minikube running

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    KUBERNETES CLUSTER                        │
│                                                              │
│  ┌────────────────────────────────────────────────────┐     │
│  │                    POD                               │     │
│  │                                                     │     │
│  │  ┌──────────────────────────────────────────┐      │     │
│  │  │              Container                     │      │     │
│  │  │                                            │      │     │
│  │  │  Environment Variables ←── ConfigMap       │      │     │
│  │  │  Environment Variables ←── Secret          │      │     │
│  │  │                                            │      │     │
│  │  │  /etc/config/ ←── ConfigMap (volume)       │      │     │
│  │  │  /etc/secrets/ ←── Secret (volume)         │      │     │
│  │  │                                            │      │     │
│  │  └──────────────────────────────────────────┘      │     │
│  └────────────────────────────────────────────────────┘     │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │  ConfigMap    │  │  Secret       │                        │
│  │  (plaintext)  │  │  (base64)    │                        │
│  │               │  │               │                        │
│  │  DB_HOST=... │  │  DB_PASS=... │                        │
│  │  DB_PORT=... │  │  API_KEY=... │                        │
│  └──────────────┘  └──────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create ConfigMap from Literal Values

```bash
# Method 1: kubectl create configmap (imperative)
kubectl create configmap app-config \
  --from-literal=DATABASE_HOST=mysql-service \
  --from-literal=DATABASE_PORT=3306 \
  --from-literal=REDIS_HOST=redis-service
```

```bash
# Check the ConfigMap
kubectl get configmap app-config
kubectl describe configmap app-config
```

Expected output:
```
Name:         app-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DATABASE_HOST:
----
mysql-service
DATABASE_PORT:
----
3306
REDIS_HOST:
----
redis-service
```

📸 **Screenshot Placeholder:** *[Terminal showing ConfigMap details]*

> 💡 **Rithu's Tip:** *"ConfigMaps store non-sensitive configuration. For passwords and API keys, use Secrets instead!"*

---

### Step 2: Create ConfigMap from a File

```bash
# Create a config file
cat > application.properties << 'EOF'
server.port=8080
server.name=my-app
logging.level=INFO
spring.profiles.active=production
EOF
```

```bash
# Create ConfigMap from file
kubectl create configmap app-config-file --from-file=application.properties
```

```bash
# Check the content
kubectl get configmap app-config-file -o yaml
```

> 💡 **Rithu's Tip:** *"When you create a ConfigMap from a file, the filename becomes the key and the file content becomes the value!"*

📸 **Screenshot Placeholder:** *[Terminal showing ConfigMap from file]*

---

### Step 3: Create ConfigMap from a Directory

```bash
# Create a directory with multiple config files
mkdir -p config-files
cat > config-files/nginx.conf << 'EOF'
server {
    listen 80;
    server_name localhost;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
    
    location /api {
        proxy_pass http://backend:8080;
    }
}
EOF
```

```bash
cat > config-files/index.html << 'EOF'
<html>
<body>
    <h1>Hello from ConfigMap!</h1>
</body>
</html>
EOF
```

```bash
# Create ConfigMap from directory
kubectl create configmap nginx-config --from-file=config-files/
```

```bash
kubectl get configmap nginx-config -o yaml
```

---

### Step 4: Create ConfigMap from YAML

```bash
cat > configmap.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-settings
data:
  APP_ENV: "production"
  APP_LOG_LEVEL: "info"
  APP_MAX_CONNECTIONS: "100"
  APP_TIMEOUT: "30"
  database.conf: |
    host=mysql-service
    port=3306
    name=myapp
    pool_size=10
EOF
```

```bash
kubectl apply -f configmap.yaml
kubectl get configmap app-settings
```

---

### Step 5: Inject ConfigMap as Environment Variables

```bash
cat > configmap-env-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: config-env-pod
spec:
  containers:
  - name: app
    image: busybox:latest
    command: ["env"]
    envFrom:
    - configMapRef:
        name: app-settings
  restartPolicy: Never
EOF
```

```bash
kubectl apply -f configmap-env-pod.yaml
kubectl logs config-env-pod
```

Expected output (you'll see all environment variables):
```
APP_ENV=production
APP_LOG_LEVEL=info
APP_MAX_CONNECTIONS=100
APP_TIMEOUT=30
KUBERNETES_SERVICE_HOST=...
...
```

📸 **Screenshot Placeholder:** *[Terminal showing env vars from ConfigMap]*

> 💡 **Rithu's Tip:** *"envFrom injects ALL keys from a ConfigMap as environment variables. If you want specific keys, use individual env entries with valueFrom!"*

---

### Step 6: Inject Specific ConfigMap Keys

```bash
cat > configmap-selective-env.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: config-selective-pod
spec:
  containers:
  - name: app
    image: busybox:latest
    command: ["env"]
    env:
    - name: MY_DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: app-settings
          key: APP_ENV
    - name: MY_LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-settings
          key: APP_LOG_LEVEL
  restartPolicy: Never
EOF
```

```bash
kubectl apply -f configmap-selective-env.yaml
kubectl logs config-selective-pod
```

Expected output:
```
MY_DATABASE_HOST=production
MY_LOG_LEVEL=info
```

---

### Step 7: Mount ConfigMap as a Volume

```bash
cat > configmap-volume-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: config-volume-pod
spec:
  containers:
  - name: app
    image: busybox:latest
    command: ["sh", "-c", "ls /etc/config/ && echo '---' && cat /etc/config/database.conf"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-settings
  restartPolicy: Never
EOF
```

```bash
kubectl apply -f configmap-volume-pod.yaml
kubectl logs config-volume-pod
```

Expected output:
```
APP_ENV
APP_LOG_LEVEL
APP_MAX_CONNECTIONS
APP_TIMEOUT
database.conf
---
host=mysql-service
port=3306
name=myapp
pool_size=10
```

> 💡 **Rithu's Tip:** *"Volume mounts are perfect for config files (nginx.conf, application.properties, etc.) while environment variables are great for simple key-value pairs!"*

📸 **Screenshot Placeholder:** *[Terminal showing ConfigMap mounted as volume]*

---

### Step 8: Create Secrets

Secrets are similar to ConfigMaps but store sensitive data (base64-encoded).

```bash
# Create Secret from literal values
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password='S3cr3tP@ss!'
```

```bash
# Check the secret
kubectl get secret db-secret
kubectl describe secret db-secret
```

Expected output:
```
Name:         db-secret
Namespace:    default
Type:         Opaque

Data
====
password:    8 bytes
username:    5 bytes
```

> ⚠️ **Note:** `kubectl describe` doesn't show secret values! It only shows the keys and their sizes.

📸 **Screenshot Placeholder:** *[Terminal showing secret description]*

---

### Step 9: Create Secret from YAML (base64 encoded)

```bash
# Encode values to base64
echo -n 'admin' | base64
# Output: YWRtaW4=

echo -n 'S3cr3tP@ss!' | base64
# Output: UzNjcjN0UEBzcyE=
```

```bash
cat > secret.yaml << 'EOF'
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=
  password: UzNjcjN0UEBzcyE=
  api-key: c2VjcmV0LWFwaS1rZXktMTIzNDU=
EOF
```

```bash
kubectl apply -f secret.yaml
kubectl get secret app-secret -o yaml
```

> 💡 **Rithu's Tip:** *"base64 is NOT encryption — it's just encoding! Anyone can decode it. For real security, use external secret managers (Vault, AWS Secrets Manager) or enable encryption at rest. Secrets only protect against casual observation."*

---

### Step 10: Decode Secrets

```bash
# Decode a secret value
kubectl get secret app-secret -o jsonpath='{.data.username}' | base64 --decode
# Output: admin

kubectl get secret app-secret -o jsonpath='{.data.password}' | base64 --decode
# Output: S3cr3tP@ss!

# Decode all values at once
kubectl get secret app-secret -o json | jq -r '.data | to_entries[] | "\(.key): \(.value | @base64)"'
```

---

### Step 11: Inject Secrets as Environment Variables

```bash
cat > secret-env-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: app
    image: busybox:latest
    command: ["env"]
    envFrom:
    - secretRef:
        name: app-secret
  restartPolicy: Never
EOF
```

```bash
kubectl apply -f secret-env-pod.yaml
kubectl logs secret-env-pod
```

Expected output:
```
api-key=secret-api-key-12345
password=S3cr3tP@ss!
username=admin
```

---

### Step 12: Mount Secret as a Volume

```bash
cat > secret-volume-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: app
    image: busybox:latest
    command: ["sh", "-c", "ls /etc/secrets/ && echo '---' && cat /etc/secrets/username"]
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
  restartPolicy: Never
EOF
```

```bash
kubectl apply -f secret-volume-pod.yaml
kubectl logs secret-volume-pod
```

> 💡 **Rithu's Tip:** *"By default, secret files have 0644 permissions. For production, set defaultMode to 0400 so only the owner can read them!"*

---

### Step 13: Real-World Example: Nginx with Config

Let's put it all together with a real nginx deployment:

```bash
cat > nginx-with-config.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
        listen 80;
        server_name localhost;
        
        location / {
            root /usr/share/nginx/html;
            index index.html;
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
  name: nginx-configured
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-configured
  template:
    metadata:
      labels:
        app: nginx-configured
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-vhost
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: nginx-vhost
        configMap:
          name: nginx-config
          items:
          - key: nginx.conf
            path: default.conf
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-configured
spec:
  type: NodePort
  selector:
    app: nginx-configured
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30090
EOF
```

```bash
kubectl apply -f nginx-with-config.yaml
kubectl get pods -l app=nginx-configured

# Test it
minikube service nginx-configured --url
curl $(minikube service nginx-configured --url)/health
# Expected: OK
```

📸 **Screenshot Placeholder:** *[Terminal showing custom nginx config working]*

---

### Step 14: Clean Up

```bash
kubectl delete -f .
kubectl delete configmap --all
kubectl delete secret --all
rm -rf config-files *.yaml *.properties
```

---

## ✅ Verification

```bash
# 1. ConfigMap creation
kubectl get configmap app-config
# Expected: ConfigMap exists

# 2. ConfigMap as env
kubectl apply -f configmap-env-pod.yaml
kubectl logs config-env-pod
# Expected: Environment variables listed

# 3. Secret creation
kubectl get secret db-secret
# Expected: Secret exists

# 4. Secret decoding
kubectl get secret db-secret -o jsonpath='{.data.username}' | base64 --decode
# Expected: admin

# 5. Cleanup
kubectl delete configmap --all
kubectl delete secret --all
kubectl delete pod --all
```

---

## 🧹 Cleanup

```bash
kubectl delete configmap --all
kubectl delete secret --all
kubectl delete pod --all
kubectl delete deploy --all
kubectl delete svc --all
rm -rf config-files *.yaml *.properties 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **ConfigMap** | Store non-sensitive configuration data |
| **Secret** | Store sensitive data (base64-encoded) |
| **envFrom** | Inject all keys as environment variables |
| **valueFrom** | Inject specific keys as environment variables |
| **Volume Mount** | Mount ConfigMaps/Secrets as files |
| **base64** | Encoding (NOT encryption) for Secret data |

---

## 🚀 What's Next?

Let's learn about persistent storage:

**[Lab 07: Persistent Volumes & Claims →](../07 - Persistent Volumes and Claims/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 ConfigMaps & Secrets mastered! Your apps are now        ║
║     configurable AND secure (well, sort of 😉).             ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
