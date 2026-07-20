# Lab 20 — Helm Charts

<div align="center">

![Lab-20](https://img.shields.io/badge/Lab-20-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Medium-purple)
![Time](https://img.shields.io/badge/Time-40%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Helm-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 20 — Helm Charts                                         ║
║  "The Package Manager for Kubernetes"                        ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"If Kubernetes is an app store, Helm is the search bar, install button, and update manager all in one. Why write 50 YAML files when you can install one chart?"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Install and use Helm
- ✅ Deploy apps from public charts
- ✅ Create your own custom chart
- ✅ Use templates and values
- ✅ Debug Helm charts

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-19
- [ ] Minikube running
- [ ] Helm installed

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    HELM ARCHITECTURE                              │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Helm Chart                              │  │
│  │                                                            │  │
│  │  Chart.yaml          ← Metadata (name, version)           │  │
│  │  values.yaml         ← Default configuration values       │  │
│  │  templates/                                                │  │
│  │  ├── deployment.yaml  ← K8s manifest with variables       │  │
│  │  ├── service.yaml     ← K8s manifest with variables       │  │
│  │  ├── ingress.yaml     ← K8s manifest with variables       │  │
│  │  └── _helpers.tpl     ← Template helpers                  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  VALUES → TEMPLATES → RENDERED MANIFESTS → KUBERNETES           │
│                                                                  │
│  values.yaml:                                                   │
│  ├── replicaCount: 3                                            │
│  ├── image: nginx:1.25                                          │
│  └── service:                                                   │
│      ├── type: ClusterIP                                        │
│      └── port: 80                                               │
│                                                                  │
│  Templates render to:                                           │
│  ├── Deployment with 3 replicas of nginx:1.25                  │
│  └── Service on port 80                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Verify Helm Installation

```bash
helm version
```

Expected output:
```
version.BuildInfo{Version:"v3.x.x"...}
```

```bash
# Add popular repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

---

### Step 2: Search for Charts

```bash
# Search for nginx charts
helm search repo nginx
```

Expected output:
```
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
bitnami/nginx                   15.x.x          1.25.x          NGINX Open Source is a web server that can be also...
bitnami/nginx-ingress-controller 10.x.x        1.9.x           NGINX Ingress Controller is an Ingress controller t...
```

> 💡 **Rithu's Tip:** *"Helm repos are like npm registries or Docker Hub — they host pre-packaged Kubernetes applications!"*

---

### Step 3: Install a Chart

```bash
# Install nginx from Bitnami chart
helm install my-nginx bitnami/nginx \
  --set service.type=NodePort \
  --set service.nodePort=30080
```

```bash
# Check the release
helm list
```

Expected output:
```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
my-nginx        default         1               2024-01-01 12:00:00 +0000 UTC           deployed        nginx-15.x.x    1.25.x
```

```bash
# Check the resources created
kubectl get pods -l app.kubernetes.io/name=nginx
kubectl get svc -l app.kubernetes.io/name=nginx
```

📸 **Screenshot Placeholder:** *[Terminal showing Helm release and resources]*

> 💡 **Rithu's Tip:** *"One helm install command created a Deployment, Service, ConfigMap, and more! That's the power of Helm packages!"*

---

### Step 4: Upgrade a Release

```bash
# Upgrade to use more replicas
helm upgrade my-nginx bitnami/nginx \
  --set replicaCount=3 \
  --set service.type=NodePort
```

```bash
# Check the upgrade
helm list
kubectl get pods -l app.kubernetes.io/name=nginx
```

Expected output:
```
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-xxxxxxxxx-xxxxx   1/1     Running   0          1m
my-nginx-yyyyyyyy-yyyyy   1/1     Running   0          1m
my-nginx-zzzzzzzz-zzzzz   1/1     Running   0          1m
```

---

### Step 5: View Release History

```bash
# See history
helm history my-nginx
```

Expected output:
```
REVISION        UPDATED                         STATUS          CHART           APP VERSION
1               Mon Jan 01 12:00:00 2024        superseded      nginx-15.x.x    1.25.x
2               Mon Jan 01 12:05:00 2024        deployed        nginx-15.x.x    1.25.x
```

```bash
# Rollback to revision 1
helm rollback my-nginx 1
helm history my-nginx
```

---

### Step 6: Create Your Own Chart

```bash
# Create a new chart
helm create my-app
```

Let's explore the structure:

```bash
find my-app -type f
```

Expected output:
```
my-app/Chart.yaml
my-app/values.yaml
my-app/templates/_helpers.tpl
my-app/templates/deployment.yaml
my-app/templates/service.yaml
my-app/templates/hpa.yaml
my-app/templates/ingress.yaml
my-app/templates/serviceaccount.yaml
my-app/templates/NOTES.txt
```

> 💡 **Rithu's Tip:** *"helm create generates a production-ready chart structure with all the best practices built in!"*

📸 **Screenshot Placeholder:** *[Terminal showing chart structure]*

---

### Step 7: Customize Your Chart

Edit `my-app/values.yaml`:

```yaml
replicaCount: 2

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.25"

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 200m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 64Mi
```

---

### Step 8: Template Syntax

Look at `my-app/templates/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.port }}
```

Key template features:
- `{{ .Values.key }}` — Access values.yaml
- `{{ .Release.Name }}` — Release metadata
- `{{ include "template" . }}` — Include helper templates
- `{{- ... | nindent 4 }}` — Pipe to functions with indentation

---

### Step 9: Install Your Custom Chart

```bash
# Lint the chart (check for errors)
helm lint my-app

# Template it (render manifests without installing)
helm template my-app my-app

# Install it!
helm install my-custom-app my-app
```

```bash
# Verify
helm list
kubectl get pods -l app.kubernetes.io/name=my-app
```

📸 **Screenshot Placeholder:** *[Terminal showing custom chart deployed]*

---

### Step 10: Debug Helm Charts

```bash
# Render templates with debug info
helm template my-app my-app --debug

# Get release values
helm get values my-custom-app

# Check release status
helm status my-custom-app

# Dry run (simulate without installing)
helm install my-dry-run my-app --dry-run --debug
```

> 💡 **Rithu's Tip:** *"helm template is your best friend for debugging! It shows you exactly what Kubernetes resources will be created before you deploy!"*

---

### Step 11: Clean Up

```bash
# Uninstall releases
helm uninstall my-nginx
helm uninstall my-custom-app

# Remove the chart
rm -rf my-app

# Verify
helm list
# Expected: No releases
```

---

## ✅ Verification

```bash
# 1. Helm install works
helm install test-nginx bitnami/nginx --set service.type=ClusterIP
helm list
# Expected: test-nginx deployed

# 2. Custom chart works
helm create test-chart
helm lint test-chart
helm install test-custom test-chart
helm list
# Expected: 2 releases

# 3. Cleanup
helm uninstall test-nginx test-custom
rm -rf test-chart
```

---

## 🧹 Cleanup

```bash
helm uninstall --all 2>/dev/null
helm list
rm -rf my-app 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Helm** | Package manager for Kubernetes |
| **Chart** | A package of pre-configured K8s resources |
| **Release** | A specific deployment of a chart |
| **values.yaml** | Configuration for chart templates |
| **Templates** | Go-based templating for K8s manifests |
| **helm create** | Generate a new chart scaffold |
| **helm lint** | Validate chart syntax |
| **helm template** | Render manifests without installing |

---

## 🚀 What's Next?

Let's add monitoring to your cluster:

**[Lab 21: Monitoring with Prometheus & Grafana →](../21 - Monitoring with Prometheus and Grafana/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Helm mastered! You can now package and deploy           ║
║     applications like a pro! 📦                             ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
