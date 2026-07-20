# Lab 04 — Deployments

<div align="center">

![Lab-04](https://img.shields.io/badge/Lab-04-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐%20Easy--Med-blue)
![Time](https://img.shields.io/badge/Time-35%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Deployments-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 04 — Deployments                                       ║
║  "Rolling Updates Without the Drama"                        ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Deployments are like GPS for your pods — they know where you're going (desired state), how to get there (rolling updates), and how to go back if you take a wrong turn (rollbacks)!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand the difference between ReplicaSets and Deployments
- ✅ Create Deployments using YAML manifests
- ✅ Perform rolling updates
- ✅ Roll back to previous versions
- ✅ Explore revision history

---

## 🧠 Prerequisites

- [ ] Completed Lab 03 (ReplicaSets)
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
│                        DEPLOYMENT                                │
│                                                                   │
│  ┌──────────────────────────────────────────────────────┐       │
│  │              ReplicaSet (v1 - nginx:1.25)             │       │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐              │       │
│  │  │ Pod 1   │  │ Pod 2   │  │ Pod 3   │              │       │
│  │  │ nginx   │  │ nginx   │  │ nginx   │  Scaling down│       │
│  │  │ 1.25    │  │ 1.25    │  │ 1.25    │  ⬇️           │       │
│  │  └─────────┘  └─────────┘  └─────────┘              │       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                   │
│  ┌──────────────────────────────────────────────────────┐       │
│  │              ReplicaSet (v2 - nginx:1.26)             │       │
│  │  ┌─────────┐  ┌─────────┐                            │       │
│  │  │ Pod 4   │  │ Pod 5   │  Scaling up ⬆️              │       │
│  │  │ nginx   │  │ nginx   │                            │       │
│  │  │ 1.26    │  │ 1.26    │                            │       │
│  │  └─────────┘  └─────────┘                            │       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                   │
│  ⏪ Revision History                                             │
│  ┌──────┐ ┌──────┐ ┌──────┐                                    │
│  │  v1  │ │  v2  │ │  v3  │  ← Rollback to any version!       │
│  └──────┘ └──────┘ └──────┘                                    │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create a Deployment

```bash
cat > deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "250m"
EOF
```

```bash
# Apply the deployment
kubectl apply -f deployment.yaml
```

```bash
# Check deployment status
kubectl get deployments
```

Expected output:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           10s
```

```bash
# Check the ReplicaSets created by the deployment
kubectl get replicasets
```

Expected output:
```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-7fb96c846b   3         3         3       15s
```

> 💡 **Rithu's Tip:** *"A Deployment automatically creates a ReplicaSet! The Deployment manages the ReplicaSet, which manages the Pods. It's like a corporate hierarchy: CEO → Manager → Workers!"*

📸 **Screenshot Placeholder:** *[Terminal showing deployment and replicaset]*

---

### Step 2: Understand Rolling Update Strategy

The key configuration in our deployment:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Max pods ABOVE desired count during update
    maxUnavailable: 1  # Max pods BELOW desired count during update
```

```
ROLLING UPDATE PROCESS (maxSurge: 1, maxUnavailable: 1):

Step 1: Create 1 new pod
  Desired: 3    Current: 4 (3 old + 1 new)
  
  [Old Pod 1] [Old Pod 2] [Old Pod 3] [NEW Pod]
  
Step 2: Terminate 1 old pod
  Desired: 3    Current: 3 (2 old + 1 new)
  
  [Old Pod 1] [Old Pod 2] [NEW Pod]
  
Step 3: Create another new pod
  Desired: 3    Current: 4 (2 old + 2 new)
  
  [Old Pod 1] [OLD Pod] [NEW Pod] [NEW Pod 2]
  
Step 4: Terminate another old pod
  Desired: 3    Current: 3 (1 old + 2 new)
  
  [OLD Pod] [NEW Pod] [NEW Pod 2]
  
Step 5: Create final new pod
  Desired: 3    Current: 4 (1 old + 3 new)

Step 6: Terminate last old pod
  Desired: 3    Current: 3 (3 new)
  
  [NEW Pod] [NEW Pod 2] [NEW Pod 3]  ✅ COMPLETE!
```

---

### Step 3: Perform a Rolling Update

Let's update from nginx:1.25 to nginx:1.26:

```bash
# Method 1: kubectl set image
kubectl set image deployment/nginx-deployment nginx=nginx:1.26
```

```bash
# Watch the rollout in real-time
kubectl rollout status deployment/nginx-deployment
```

You'll see:
```
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

📸 **Screenshot Placeholder:** *[Terminal showing rollout status]*

> 💡 **Rithu's Tip:** *"Rolling updates ensure ZERO downtime! New pods are created before old ones are terminated. Your users won't even notice the update happening!"*

---

### Step 4: Check the Deployment After Update

```bash
# Check the deployment
kubectl get deployments
```

Expected output:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           5m
```

```bash
# Check the ReplicaSets - you should see TWO now!
kubectl get replicasets
```

Expected output:
```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-5fb9c5db8f   3         3         3       1m    ← v2 (nginx:1.26)
nginx-deployment-7fb96c846b   0         0         0       6m    ← v1 (nginx:1.25) - scaled to 0
```

> 💡 **Rithu's Tip:** *"The old ReplicaSet is kept around with 0 replicas. This is important — it's how Kubernetes can roll back! If you deleted it, you couldn't go back to v1."*

```bash
# Verify the image version on running pods
kubectl get pods -l app=nginx -o jsonpath='{.items[*].spec.containers[*].image}'
```

Expected output:
```
nginx:1.26 nginx:1.26 nginx:1.26
```

---

### Step 5: View Revision History

```bash
# See the deployment's revision history
kubectl rollout history deployment/nginx-deployment
```

Expected output:
```
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

```bash
# See details of a specific revision
kubectl rollout history deployment/nginx-deployment --revision=1
```

This shows you the full pod template of revision 1, including the image nginx:1.25.

> 💡 **Rithu's Tip:** *"By default, Kubernetes doesn't annotate what changed. You can add --record to capture the command, but in modern K8s, you should use annotations for tracking changes!"*

---

### Step 6: Roll Back to Previous Version

Let's say the nginx:1.26 update has a bug and you need to go back:

```bash
# Rollback to previous revision
kubectl rollout undo deployment/nginx-deployment
```

```bash
# Watch the rollback
kubectl rollout status deployment/nginx-deployment
```

Expected output:
```
deployment "nginx-deployment" successfully rolled out
```

```bash
# Verify we're back to nginx:1.25
kubectl get pods -l app=nginx -o jsonpath='{.items[*].spec.containers[*].image}'
```

Expected output:
```
nginx:1.25 nginx:1.25 nginx:1.25
```

📸 **Screenshot Placeholder:** *[Terminal showing successful rollback]*

> 💡 **Rithu's Tip:** *"Rollbacks are the 'undo' button of Kubernetes. Use it wisely! In production, you'd also have monitoring to catch issues before they affect all users."*

---

### Step 7: Roll Back to Specific Revision

```bash
# Check current history
kubectl rollout history deployment/nginx-deployment
```

Expected output:
```
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

```bash
# Roll back to revision 2
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

```bash
# Verify
kubectl get pods -l app=nginx -o jsonpath='{.items[*].spec.containers[*].image}'
```

---

### Step 8: Update Deployment via YAML

Let's update the deployment properly using YAML:

```bash
cat > deployment-v2.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "250m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
EOF
```

```bash
# Apply the updated deployment
kubectl apply -f deployment-v2.yaml
```

```bash
# Watch the rollout
kubectl rollout status deployment/nginx-deployment
```

```bash
# Verify 5 pods running with nginx:1.27
kubectl get pods -l app=nginx
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

> 💡 **Rithu's Tip:** *"Notice the readinessProbe and livenessProbe? ReadinessProbe checks if the pod is ready to receive traffic. LivenessProbe checks if the pod is alive. If liveness fails, Kubernetes restarts the pod!"*

📸 **Screenshot Placeholder:** *[Terminal showing 5 pods with nginx:1.27]*

---

### Step 9: Scale the Deployment

```bash
# Scale down
kubectl scale deployment nginx-deployment --replicas=2
kubectl get deployment nginx-deployment
```

Expected output:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           10m
```

```bash
# Scale back up
kubectl scale deployment nginx-deployment --replicas=4
kubectl get deployment nginx-deployment
```

Expected output:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   4/4     4            4           10m
```

---

### Step 10: Deployment Pausing and Resuming

```bash
# Pause a rollout (useful for batch updates)
kubectl rollout pause deployment/nginx-deployment

# Make multiple changes
kubectl set image deployment/nginx-deployment nginx=nginx:1.28
kubectl set resources deployment/nginx-deployment -c nginx --limits=cpu=500m,memory=256Mi

# Check status — rollout is paused, so changes haven't been applied
kubectl rollout status deployment/nginx-deployment
# You'll see: "deployment "nginx-deployment" is paused"

# Resume the rollout (applies all changes at once)
kubectl rollout resume deployment/nginx-deployment

# Watch it complete
kubectl rollout status deployment/nginx-deployment
```

> 💡 **Rithu's Tip:** *"Pausing is great when you want to make several changes at once. Instead of triggering 3 separate rollouts, you pause, make all changes, then resume — one clean rollout!"*

---

### Step 11: Clean Up

```bash
# Delete the deployment
kubectl delete -f deployment.yaml
kubectl delete -f deployment-v2.yaml 2>/dev/null
kubectl delete deployment nginx-deployment 2>/dev/null

# Verify
kubectl get all
# Expected: No resources found in default namespace.
```

```bash
rm deployment.yaml deployment-v2.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. Create deployment
kubectl apply -f deployment.yaml
kubectl get deployments
# Expected: nginx-deployment 3/3

# 2. Rolling update works
kubectl set image deployment/nginx-deployment nginx=nginx:1.26
kubectl rollout status deployment/nginx-deployment
# Expected: deployment successfully rolled out

# 3. Rollback works
kubectl rollout undo deployment/nginx-deployment
kubectl get pods -l app=nginx -o jsonpath='{.items[0].spec.containers[0].image}'
# Expected: nginx:1.25

# 4. History works
kubectl rollout history deployment/nginx-deployment
# Expected: Multiple revisions listed

# 5. Cleanup
kubectl delete deployment nginx-deployment
```

---

## 🧹 Cleanup

```bash
kubectl delete deployment nginx-deployment
kubectl delete all --all
rm deployment*.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Deployment** | Higher-level abstraction over ReplicaSets |
| **Rolling Update** | Zero-downtime deployment strategy |
| **maxSurge** | Maximum pods above desired count during update |
| **maxUnavailable** | Maximum pods below desired count during update |
| **Rollback** | Revert to previous deployment version |
| **Revision History** | Track all deployment changes |
| **Readiness Probe** | Check if pod is ready for traffic |
| **Liveness Probe** | Check if pod is alive |
| **Pausing** | Batch multiple changes into one rollout |

---

## 🚀 What's Next?

Now that you can deploy applications, let's learn how to expose them to the network:

**[Lab 05: Services & Networking →](../05 - Services and Networking/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Rolling updates, rollbacks, probes...                   ║
║     You can now deploy like a pro! 🚀                       ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
