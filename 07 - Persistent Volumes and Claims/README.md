# Lab 07 — Persistent Volumes & Claims

<div align="center">

![Lab-07](https://img.shields.io/badge/Lab-07-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐%20Medium-blue)
![Time](https://img.shields.io/badge/Time-35%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-PV%20%2F%20PVC-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 07 — Persistent Volumes & Claims                       ║
║  "Because Pods Are Ephemeral, But Data Shouldn't Be"        ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Pods come and go, like visitors at a coffee shop. But what if you need the coffee to stay? That's where Persistent Volumes come in — they make sure your data survives even when pods don't!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand the PV/PVC lifecycle
- ✅ Create PersistentVolumes and PersistentVolumeClaims
- ✅ Use hostPath and local storage
- ✅ Understand dynamic provisioning basics
- ✅ Connect storage to pods

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-06
- [ ] Minikube running

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      STORAGE ARCHITECTURE                        │
│                                                                  │
│  ┌─────────────────────────────────────────────────────┐       │
│  │                   POD                                │       │
│  │  ┌──────────────────────────────────────────────┐   │       │
│  │  │  Container                                    │   │       │
│  │  │  Mount Path: /data                           │   │       │
│  │  └──────────────────┬───────────────────────────┘   │       │
│  └─────────────────────┼───────────────────────────────┘       │
│                        │                                        │
│  ┌─────────────────────┼───────────────────────────────┐       │
│  │           PersistentVolumeClaim (PVC)                │       │
│  │           "I need 5Gi of storage"                    │       │
│  └─────────────────────┼───────────────────────────────┘       │
│                        │                                        │
│  ┌─────────────────────┼───────────────────────────────┐       │
│  │           PersistentVolume (PV)                       │       │
│  │           "I have 10Gi of hostPath storage"          │       │
│  │           ┌─────────────────────────────────┐       │       │
│  │           │  /mnt/data (on the node)         │       │       │
│  │           └─────────────────────────────────┘       │       │
│  └─────────────────────────────────────────────────────┘       │
│                                                                  │
│  Binding: PVC requests → PV provides                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Understand the Storage Hierarchy

```
StorageClass (optional) → defines HOW to provision storage
    ↓
PersistentVolume (PV) → defines available storage
    ↓
PersistentVolumeClaim (PVC) → requests storage
    ↓
Pod → uses the PVC
```

---

### Step 2: Create a PersistentVolume (hostPath)

```bash
cat > persistent-volume.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
  labels:
    type: local
spec:
  storageClassName: standard
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
    type: DirectoryOrCreate
EOF
```

```bash
kubectl apply -f persistent-volume.yaml
kubectl get persistentvolume
```

Expected output:
```
NAME    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM   STORAGECLASS   REASON   AGE
my-pv   5Gi        RWO            Retain           Available         standard                10s
```

> 💡 **Rithu's Tip:** *"hostPath mounts a directory from the node's filesystem into the pod. It's great for learning but NOT for production — what happens if the node dies?"*

📸 **Screenshot Placeholder:** *[Terminal showing PersistentVolume]*

---

### Step 3: Create a PersistentVolumeClaim

```bash
cat > persistent-volume-claim.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
EOF
```

```bash
kubectl apply -f persistent-volume-claim.yaml
kubectl get persistentvolumeclaim
```

Expected output:
```
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pvc   Bound    my-pv    5Gi        RWO            standard       10s
```

```bash
# Check the PV is now bound
kubectl get persistentvolume
```

Expected output:
```
NAME    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS   REASON   AGE
my-pv   5Gi        RWO            Retain           Bound    my-pvc/my-pvc  standard                2m
```

> 💡 **Rithu's Tip:** *"Notice the PV was 5Gi but the PVC only asked for 2Gi? That's fine! The PV must be equal to or larger than the PVC request. Think of it like renting an apartment — you can't rent a 3-bedroom when you only need 1!"*

📸 **Screenshot Placeholder:** *[Terminal showing PVC bound to PV]*

---

### Step 4: Use the PVC in a Pod

```bash
cat > pv-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: pv-demo-pod
spec:
  containers:
  - name: app
    image: busybox:latest
    command: ["sh", "-c", "echo 'Hello from PV!' > /data/hello.txt && cat /data/hello.txt && sleep 3600"]
    volumeMounts:
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: my-pvc
EOF
```

```bash
kubectl apply -f pv-pod.yaml
kubectl get pods pv-demo-pod
```

Wait for Running, then:

```bash
# Verify data was written
kubectl exec pv-demo-pod -- cat /data/hello.txt
# Expected: Hello from PV!
```

📸 **Screenshot Placeholder:** *[Terminal showing data written to PV]*

---

### Step 5: Test Data Persistence

Let's see if data survives pod deletion:

```bash
# Delete the pod
kubectl delete pod pv-demo-pod
```

```bash
# Create a new pod with the same PVC
cat > pv-pod-2.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: pv-demo-pod-2
spec:
  containers:
  - name: app
    image: busybox:latest
    command: ["sh", "-c", "cat /data/hello.txt && echo 'Data survived!' && sleep 3600"]
    volumeMounts:
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: my-pvc
EOF
```

```bash
kubectl apply -f pv-pod-2.yaml
kubectl logs pv-demo-pod-2
```

Expected output:
```
Hello from PV!
Data survived!
```

> 💡 **Rithu's Tip:** *"That's the magic of PV/PVC! The data persisted even though the pod was destroyed. The PV lives independently of any pod."*

📸 **Screenshot Placeholder:** *[Terminal showing data survived pod deletion]*

---

### Step 6: Access Modes

PVs support three access modes:

| Access Mode | Abbreviation | Description |
|-------------|--------------|-------------|
| ReadWriteOnce | RWO | Mounted as read-write by a single node |
| ReadOnlyMany | ROX | Mounted as read-only by many nodes |
|ReadWriteMany | RWX | Mounted as read-write by many nodes |

```bash
# Create a ReadWriteMany PV (for learning - not all backends support it)
cat > pv-rwx.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolume
metadata:
  name: shared-pv
spec:
  storageClassName: standard
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/mnt/shared"
    type: DirectoryOrCreate
EOF
```

> 💡 **Rithu's Tip:** *"hostPath only supports ReadWriteOnce. For ReadWriteMany, you'd need NFS, CephFS, or cloud storage. In production, most stateful apps use RWO!"*

---

### Step 7: Reclaim Policies

When a PVC is deleted, what happens to the PV?

| Policy | Behavior |
|--------|----------|
| **Retain** | PV and data are kept (manual cleanup required) |
| **Delete** | PV and underlying storage are deleted |
| **Recycle** | Deprecated - cleared data and made available again |

```bash
# Check the reclaim policy on our PV
kubectl get pv my-pv -o jsonpath='{.spec.persistentVolumeReclaimPolicy}'
# Expected: Retain
```

---

### Step 8: StorageClasses and Dynamic Provisioning

Instead of manually creating PVs, StorageClasses allow automatic provisioning:

```bash
# Check available StorageClasses
kubectl get storageclass
```

Expected output:
```
NAME                 PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath       Delete          Immediate           true                   30m
```

```bash
cat > dynamic-pvc.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF
```

```bash
kubectl apply -f dynamic-pvc.yaml
kubectl get pvc dynamic-pvc
```

Expected output:
```
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
dynamic-pvc   Bound    pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   1Gi        RWO            standard       10s
```

> 💡 **Rithu's Tip:** *"The StorageClass automatically created a PV for us! No need to manually create one. This is how most production clusters work!"*

📸 **Screenshot Placeholder:** *[Terminal showing dynamically provisioned PVC]*

---

### Step 9: Clean Up

```bash
# Delete pods
kubectl delete pod --all

# Delete PVCs
kubectl delete pvc --all

# Delete PVs
kubectl delete pv --all

# Verify
kubectl get pv,pvc
```

```bash
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. PV creation
kubectl apply -f persistent-volume.yaml
kubectl get pv my-pv
# Expected: Available

# 2. PVC binding
kubectl apply -f persistent-volume-claim.yaml
kubectl get pvc my-pvc
# Expected: Bound

# 3. Data persistence
kubectl apply -f pv-pod.yaml
kubectl exec pv-demo-pod -- cat /data/hello.txt
# Expected: Hello from PV!

# 4. Dynamic provisioning
kubectl apply -f dynamic-pvc.yaml
kubectl get pvc dynamic-pvc
# Expected: Bound

# 5. Cleanup
kubectl delete pv --all
kubectl delete pvc --all
```

---

## 🧹 Cleanup

```bash
kubectl delete pod --all
kubectl delete pvc --all
kubectl delete pv --all
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **PersistentVolume (PV)** | A piece of storage in the cluster |
| **PersistentVolumeClaim (PVC)** | A request for storage by a pod |
| **StorageClass** | Defines how to provision storage dynamically |
| **Access Modes** | RWO, ROX, RWX — who can mount the storage |
| **Reclaim Policy** | What happens when PVC is deleted |
| **Dynamic Provisioning** | Automatic PV creation via StorageClasses |

---

## 🚀 What's Next?

Now let's use PV/PVC with StatefulSets for stateful applications:

**[Lab 08: StatefulSets →](../08 - StatefulSets/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Your data now survives pod crashes!                     ║
║     That's a big step toward production-ready apps! 📦      ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
