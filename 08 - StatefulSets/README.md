# Lab 08 — StatefulSets

<div align="center">

![Lab-08](https://img.shields.io/badge/Lab-08-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐%20Medium-purple)
![Time](https://img.shields.io/badge/Time-40%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-StatefulSets-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 08 — StatefulSets                                       ║
║  "For When Pods Need Names and Stability"                    ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Deployments are great for stateless apps. But databases? They need stable names, stable storage, and ordered operations. That's what StatefulSets are for — they give pods an identity!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand the difference between Deployments and StatefulSets
- ✅ Deploy MySQL with a StatefulSet
- ✅ Observe ordered pod creation and deletion
- ✅ Understand stable network identity
- ✅ Use Headless Services with StatefulSets

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-07
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
│                     STATEFULSET ARCHITECTURE                      │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Headless Service (mysql-headless)            │   │
│  │              clusterIP: None                              │   │
│  └──────────┬───────────────┬───────────────┬───────────────┘   │
│             │               │               │                    │
│  ┌──────────▼──────┐ ┌─────▼──────────┐ ┌──▼───────────────┐  │
│  │    mysql-0      │ │    mysql-1      │ │    mysql-2        │  │
│  │                 │ │                 │ │                   │  │
│  │ Stable Name:    │ │ Stable Name:    │ │ Stable Name:      │  │
│  │ mysql-0.mysql   │ │ mysql-1.mysql   │ │ mysql-2.mysql     │  │
│  │ .headless.svc   │ │ .headless.svc   │ │ .headless.svc     │  │
│  │                 │ │                 │ │                   │  │
│  │ ┌───────────┐  │ │ ┌───────────┐  │ │ ┌───────────┐    │  │
│  │ │ mysql-0   │  │ │ │ mysql-1   │  │ │ │ mysql-2   │    │  │
│  │ │ (data)    │  │ │ │ (data)    │  │ │ │ (data)    │    │  │
│  │ └───────────┘  │ │ └───────────┘  │ │ └───────────┘    │  │
│  └────────────────┘ └────────────────┘ └───────────────────┘  │
│                                                                   │
│  Creation Order: mysql-0 → mysql-1 → mysql-2 (sequential)       │
│  Deletion Order: mysql-2 → mysql-1 → mysql-0 (reverse)          │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create a Namespace (Optional but Good Practice)

```bash
kubectl create namespace statefulset-demo
kubectl config set-context --current --namespace=statefulset-demo
```

---

### Step 2: Create the Headless Service

StatefulSets require a Headless Service for stable DNS entries:

```bash
cat > mysql-headless-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
EOF
```

```bash
kubectl apply -f mysql-headless-service.yaml
```

> 💡 **Rithu's Tip:** *"clusterIP: None makes it a Headless Service. Instead of a single IP, DNS returns individual pod IPs!"*

---

### Step 3: Create the StatefulSet

```bash
cat > mysql-statefulset.yaml << 'EOF'
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: standard
      resources:
        requests:
          storage: 1Gi
EOF
```

```bash
# First, create the secret
kubectl create secret generic mysql-secret \
  --from-literal=root-password='MyS3cr3t!'
```

```bash
# Apply the StatefulSet
kubectl apply -f mysql-statefulset.yaml
```

```bash
# Watch pods being created in order
kubectl get pods -w
```

Expected output (note the ordered creation!):
```
NAME      READY   STATUS    RESTARTS   AGE
mysql-0   0/1     Pending   0          0s
mysql-0   0/1     ContainerCreating   0   5s
mysql-0   1/1     Running   0          30s    ← mysql-0 must be ready first
mysql-1   0/1     Pending   0          30s    ← then mysql-1 starts
mysql-1   0/1     ContainerCreating   0   35s
mysql-1   1/1     Running   0          60s    ← then mysql-2
mysql-2   0/1     Pending   0          60s
mysql-2   1/1     Running   0          90s
```

📸 **Screenshot Placeholder:** *[Terminal showing ordered pod creation]*

> 💡 **Rithu's Tip:** *"See how each pod starts only after the previous one is ready? That's the 'Ordered' in StatefulSets! Unlike Deployments which launch all pods at once."*

---

### Step 4: Explore Stable Network Identity

```bash
# Check the StatefulSet
kubectl get statefulset
```

Expected output:
```
NAME    READY   AGE
mysql   3/3     2m
```

```bash
# Check the Headless Service endpoints
kubectl get endpoints mysql-headless
```

Expected output:
```
NAME             ENDPOINTS                                               AGE
mysql-headless   172.17.0.2:3306,172.17.0.3:3306,172.17.0.4:3306       3m
```

```bash
# Test DNS resolution
kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- nslookup mysql-headless
```

Expected output:
```
Name:      mysql-headless
Address:   172.17.0.2
Address:   172.17.0.3
Address:   172.17.0.4
```

> 💡 **Rithu's Tip:** *"Each pod gets a predictable DNS entry: <pod-name>.<headless-service>.<namespace>.svc.cluster.local. So mysql-0.mysql-headless.statefulset-demo.svc.cluster.local always points to mysql-0!"*

📸 **Screenshot Placeholder:** *[Terminal showing DNS resolution for StatefulSet pods]*

---

### Step 5: Check Volume Claims

```bash
# StatefulSets create PVCs automatically!
kubectl get pvc
```

Expected output:
```
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-data-0   Bound    pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   1Gi        RWO            standard       2m
mysql-data-1   Bound    pvc-yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy   1Gi        RWO            standard       2m
mysql-data-2   Bound    pvc-zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzz   1Gi        RWO            standard       2m
```

> 💡 **Rithu's Tip:** *"The PVC name matches the pod name (mysql-data-0, mysql-data-1, mysql-data-2). This ensures each pod gets its own dedicated storage!"*

---

### Step 6: Ordered Deletion

Let's delete the StatefulSet and observe the order:

```bash
# Scale down from 3 to 1
kubectl scale statefulset mysql --replicas=1
kubectl get pods -w
```

Expected output:
```
NAME      READY   STATUS        AGE
mysql-0   1/1     Running       5m
mysql-1   1/1     Terminating   5m    ← mysql-2 deleted first (reverse order)
mysql-2   0/1     Terminating   5m
```

```bash
# Scale back up
kubectl scale statefulset mysql --replicas=3
kubectl get pods -w
```

Expected output:
```
NAME      READY   STATUS              AGE
mysql-0   1/1     Running             6m
mysql-1   0/1     ContainerCreating   5s    ← mysql-1 created first (forward order)
mysql-2   0/1     Pending             0s    ← then mysql-2
```

> 💡 **Rithu's Tip:** *"Creation: forward order (0, 1, 2). Deletion: reverse order (2, 1, 0). This is crucial for databases — you want to shut down replicas before the primary!"*

📸 **Screenshot Placeholder:** *[Terminal showing ordered deletion]*

---

### Step 7: Verify Stable Storage Persistence

```bash
# Write data to mysql-0
kubectl exec mysql-0 -- mysql -uroot -p'MyS3cr3t!' -e "CREATE DATABASE testdb; USE testdb; CREATE TABLE hello (msg VARCHAR(50)); INSERT INTO hello VALUES ('Hello from mysql-0!');"
```

```bash
# Delete mysql-0
kubectl delete pod mysql-0
kubectl get pods -w
```

Wait for mysql-0 to come back:

```bash
# Verify data survived!
kubectl exec mysql-0 -- mysql -uroot -p'MyS3cr3t!' -e "SELECT * FROM testdb.hello;"
```

Expected output:
```
+-------------------+
| msg               |
+-------------------+
| Hello from mysql-0|
+-------------------+
```

> 💡 **Rithu's Tip:** *"The data survived because mysql-0's PVC was preserved even when the pod was deleted. When the new mysql-0 pod started, it reattached to the same PVC!"*

📸 **Screenshot Placeholder:** *[Terminal showing data survived pod deletion]*

---

### Step 8: Full Deletion

```bash
# Delete the StatefulSet
kubectl delete statefulset mysql
kubectl get pods
kubectl get pvc
```

Note: PVCs are NOT automatically deleted when the StatefulSet is deleted! This protects your data.

```bash
# Manually delete PVCs if you want
kubectl delete pvc --all
```

---

### Step 9: Clean Up

```bash
kubectl delete -f .
kubectl delete secret mysql-secret
kubectl delete namespace statefulset-demo
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. StatefulSet creation
kubectl get statefulset mysql
# Expected: 3/3 READY

# 2. Ordered creation
kubectl get pods
# Expected: mysql-0, mysql-1, mysql-2 all Running

# 3. Stable DNS
kubectl exec dns-test -- nslookup mysql-headless
# Expected: 3 IP addresses returned

# 4. PVC creation
kubectl get pvc
# Expected: 3 PVCs (mysql-data-0, mysql-data-1, mysql-data-2)

# 5. Cleanup
kubectl delete statefulset mysql
kubectl delete pvc --all
```

---

## 🧹 Cleanup

```bash
kubectl delete statefulset mysql
kubectl delete pvc --all
kubectl delete secret mysql-secret
kubectl delete svc --all
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **StatefulSet** | Workload for stateful applications with stable identity |
| **Headless Service** | Provides individual pod DNS entries |
| **Ordered Operations** | Pods created/deleted in sequence |
| **Stable Network Identity** | Predictable DNS names (pod-name.service) |
| **VolumeClaimTemplates** | Automatically creates PVCs for each pod |
| **Data Persistence** | PVCs survive pod deletion |

---

## 🚀 What's Next?

Let's expose services to the outside world with Ingress:

**[Lab 09: Ingress Controller →](../09 - Ingress Controller/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 StatefulSets mastered! Databases on Kubernetes          ║
║     no longer seem scary, right? 🗄️                         ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
