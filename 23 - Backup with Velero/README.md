# Lab 23 — Backup with Velero

<div align="center">

![Lab-23](https://img.shields.io/badge/Lab-23-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Med--Hard-orange)
![Time](https://img.shields.io/badge/Time-35%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Velero-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 23 — Backup with Velero                                 ║
║  "Disaster Recovery for Kubernetes"                          ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Hope for the best, prepare for the worst. Velero is your 'undo' button for Kubernetes — it backs up everything and restores when things go sideways!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand Velero's backup/restore capabilities
- ✅ Install Velero on Minikube
- ✅ Create on-demand backups
- ✅ Restore from backup
- ✅ Understand scheduled backups

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-22
- [ ] Minikube running

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
☁️  In production, requires object storage (S3, GCS, Azure Blob)
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    VELERO ARCHITECTURE                           │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    BACKUP PROCESS                          │  │
│  │                                                            │  │
│  │  ┌─────────┐    ┌─────────┐    ┌──────────────────┐     │  │
│  │  │ velero  │───▶│  K8s    │───▶│ Object Storage   │     │  │
│  │  │ (backup)│    │  API    │    │ (S3/GCS/local)   │     │  │
│  │  └─────────┘    └─────────┘    └──────────────────┘     │  │
│  │                                                            │  │
│  │  Backs up:                                                │  │
│  │  ├── Namespaces                                          │  │
│  │  ├── Deployments, Services, ConfigMaps                   │  │
│  │  ├── PV data (via snapshots)                             │  │
│  │  └── Custom resources                                    │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    RESTORE PROCESS                         │  │
│  │                                                            │  │
│  │  ┌─────────┐    ┌─────────┐    ┌──────────────────┐     │  │
│  │  │ velero  │◀───│ Object  │◀───│ K8s API          │     │  │
│  │  │(restore)│    │ Storage │    │ (recreate resources)│   │  │
│  │  └─────────┘    └─────────┘    └──────────────────┘     │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Install Velero CLI

```bash
# Download Velero CLI
curl -sSL https://github.com/vmware-tanzu/velero/releases/download/v1.13.0/velero-v1.13.0-linux-amd64.tar.gz | tar -xz
sudo mv velero-v1.13.0-linux-amd64/velero /usr/local/bin/

# Verify
velero version --client-only
```

> 💡 **Rithu's Tip:** *"Velero needs two parts: the CLI (velero command) and the server (runs in your cluster as a deployment)."*

---

### Step 2: Install Velero with Local Backend

For Minikube, we'll use a local file system backup (not recommended for production):

```bash
# Create backup storage location directory
mkdir -p /tmp/velero-backups

# Install Velero server
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.7.0 \
  --bucket velero-backups \
  --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://minio.velero:9000 \
  --snapshot-location-config region=minio \
  --no-secret
```

> 💡 **Rithu's Tip:** *"For learning purposes, we're skipping the storage backend setup. In production, you'd use AWS S3, GCS, or Azure Blob Storage!"*

---

### Step 3: Verify Velero Installation

```bash
# Check Velero pods
kubectl get pods -n velero
```

Expected output:
```
NAME                      READY   STATUS    RESTARTS   AGE
velero-xxxxxxxxx-xxxxx   1/1     Running   0          1m
```

```bash
# Check Velero version
velero version
```

Expected output:
```
Client:
        Version: v1.13.0
Server:
        Version: v1.13.0
```

📸 **Screenshot Placeholder:** *[Terminal showing Velero running]*

---

### Step 4: Create Test Applications

```bash
cat > test-apps.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: backup-demo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: backup-demo
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
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: backup-demo
data:
  database_host: "mysql-backup.local"
  app_version: "1.0.0"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: backup-demo
spec:
  selector:
    app: nginx
  ports:
  - port: 80
EOF
```

```bash
kubectl apply -f test-apps.yaml
kubectl get all -n backup-demo
```

---

### Step 5: Create an On-Demand Backup

```bash
# Create a backup of the backup-demo namespace
velero backup create backup-demo-$(date +%Y%m%d) \
  --include-namespaces backup-demo
```

```bash
# Check backup status
velero backup get
```

Expected output:
```
NAME                    STATUS      ERRORS   WARNINGS   CREATED                         EXPIRES   STORAGE LOCATION   SELECTOR
backup-demo-20240101    Completed   0        0          2024-01-01 12:00:00 +0000 UTC    30d       default            <none>
```

📸 **Screenshot Placeholder:** *[Terminal showing backup completed]*

> 💡 **Rithu's Tip:** *"Always check the STATUS column! 'Completed' means success. 'PartiallyFailed' means something went wrong — check velero backup logs for details!"*

---

### Step 6: Describe the Backup

```bash
velero backup describe backup-demo-20240101
```

Expected output:
```
Name:         backup-demo-20240101
Namespace:    velero
Labels:       ...
Annotations:  ...
Phase:        Completed
Errors:       0
Warnings:     0

Namespaces:
  Included:  backup-demo
  Excluded:  <none>

Resources:
  Included:        * (all)
  Excluded:        <none>

Backup Volumes:
  Velero Backups:  1
```

---

### Step 7: Simulate Disaster — Delete Everything!

```bash
# Delete the entire namespace
kubectl delete namespace backup-demo

# Verify it's gone
kubectl get ns backup-demo
# Expected: Error from server (NotFound)
```

---

### Step 8: Restore from Backup

```bash
# List backups
velero backup get

# Restore the backup
velero restore create --from-backup backup-demo-20240101
```

```bash
# Watch the restore progress
velero restore get
```

Wait for STATUS to show `Completed`:
```
NAME                    BACKUP                  STATUS      ERRORS   WARNINGS   STARTS                         COMPLETED
restore-20240101        backup-demo-20240101    Completed   0        0          2024-01-01 12:05:00 +0000 UTC   2024-01-01 12:06:00 +0000 UTC
```

```bash
# Verify everything is restored
kubectl get all -n backup-demo
```

Expected output:
```
NAME                              READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-xxxxx-xxxxx     1/1     Running   0          30s
pod/nginx-deploy-yyyyy-yyyyy     1/1     Running   0          30s
pod/nginx-deploy-zzzzz-zzzzz     1/1     Running   0          30s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/nginx-svc    ClusterIP   10.xxx.xxx.xxx  <none>        80/TCP    30s
```

> 💡 **Rithu's Tip:** *"Everything is back! The namespace, deployment, service, and configmap — all restored from the backup!"*

📸 **Screenshot Placeholder:** *[Terminal showing restored resources]*

---

### Step 9: Scheduled Backups

```bash
# Create a scheduled backup (every 6 hours)
velero schedule create hourly-backup \
  --schedule="0 */6 * * *" \
  --include-namespaces backup-demo \
  --ttl 720h
```

```bash
# Check schedules
velero schedule get
```

Expected output:
```
NAME              STATUS    SCHEDULE      LAST BACKUP   AGE   VALID
hourly-backup     Enabled   0 */6 * * *   <none>        10s   true
```

> 💡 **Rithu's Tip:** *"Scheduled backups are essential for production! Set them up and forget about them — Velero handles the rest!"*

---

### Step 10: Clean Up

```bash
# Delete test resources
kubectl delete namespace backup-demo

# Delete Velero backups
velero backup delete --all

# Delete schedules
velero schedule delete --all

# Uninstall Velero
velero uninstall

rm -rf velero-v1.13.0-linux-amd64 *.yaml
```

---

## ✅ Verification

```bash
# 1. Velero installed
velero version
kubectl get pods -n velero

# 2. Backup created
velero backup get
# Expected: backup-demo-20240101 Completed

# 3. Restore works
kubectl get all -n backup-demo
# Expected: All resources restored

# 4. Cleanup
velero uninstall
```

---

## 🧹 Cleanup

```bash
velero backup delete --all 2>/dev/null
velero schedule delete --all 2>/dev/null
velero uninstall 2>/dev/null
kubectl delete namespace backup-demo 2>/dev/null
kubectl delete namespace velero 2>/dev/null
rm -rf velero-* *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Velero** | Kubernetes backup and disaster recovery tool |
| **On-demand Backup** | Manually trigger a backup |
| **Scheduled Backup** | Automatically backup on a schedule |
| **Restore** | Recover resources from a backup |
| **Backup Storage** | Where backups are stored (S3, GCS, local) |
| **Namespace Backup** | Backup specific namespaces |

---

## 🚀 What's Next?

Time for cloud deployments:

**[Lab 24: EKS Cluster Setup →](../24 - EKS Cluster Setup/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Section 5 Complete! Your production stack now has:       ║
║     Helm, Monitoring, Logging, and Backup! 🏭              ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
