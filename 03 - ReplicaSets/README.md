# Lab 03 — ReplicaSets

<div align="center">

![Lab-03](https://img.shields.io/badge/Lab-03-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐%20Beginner-brightgreen)
![Time](https://img.shields.io/badge/Time-25%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-ReplicaSets-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 03 — ReplicaSets                                       ║
║  "Because One Replica is Never Enough"                      ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"What happens when your one and only pod decides to take a nap? Your app goes down. ReplicaSets ensure you always have enough pods running — like a bouncer who always makes sure the club is at capacity!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand what a ReplicaSet is and why it exists
- ✅ Create ReplicaSets using YAML manifests
- ✅ Scale replicas up and down
- ✅ See self-healing in action (delete a pod → watch it come back)
- ✅ Understand the relationship between ReplicaSets and Deployments

---

## 🧠 Prerequisites

- [ ] Completed Lab 02 (Pods Deep Dive)
- [ ] Minikube running with kubectl configured

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
⏱️  All resources are local — no charges!
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    REPLICA SET                                   │
│                                                                  │
│  Desired State: "Always have 3 replicas running"                │
│                                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                      │
│  │  Pod 1   │  │  Pod 2   │  │  Pod 3   │                      │
│  │ (nginx)  │  │ (nginx)  │  │ (nginx)  │                      │
│  │ ✅ Running│  │ ✅ Running│  │ ✅ Running│                      │
│  └──────────┘  └──────────┘  └──────────┘                      │
│                                                                  │
│  If Pod 2 dies...                                               │
│                                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                      │
│  │  Pod 1   │  │  Pod 2   │  │  Pod 3   │                      │
│  │ (nginx)  │  │ (nginx)  │  │ (nginx)  │                      │
│  │ ✅ Running│  │ ❌ Died  │  │ ✅ Running│                      │
│  └──────────┘  └──────────┘  └──────────┘                      │
│                                                                  │
│  ReplicaSet Controller notices...                                │
│                                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │  Pod 1   │  │  Pod 2   │  │  Pod 3   │  │  Pod 4   │       │
│  │ (nginx)  │  │ (nginx)  │  │ (nginx)  │  │ (nginx)  │       │
│  │ ✅ Running│  │ 🔄 New!  │  │ ✅ Running│  │ ✅ Running│       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
│                                                                  │
│  Back to 3... wait, that's 4. It'll scale back down.            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create a ReplicaSet

Create a file called `replicaset.yaml`:

```bash
cat > replicaset.yaml << 'EOF'
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        tier: frontend
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
# Apply the ReplicaSet
kubectl apply -f replicaset.yaml
```

```bash
# Check the ReplicaSet
kubectl get replicaset
```

Expected output:
```
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   3         3         3       10s
```

```bash
# Check the pods created by the ReplicaSet
kubectl get pods -l app=nginx
```

Expected output:
```
NAME             READY   STATUS    RESTARTS   AGE
nginx-rs-xxxxx   1/1     Running   0          10s
nginx-rs-yyyyy   1/1     Running   0          10s
nginx-rs-zzzzz   1/1     Running   0          10s
```

📸 **Screenshot Placeholder:** *[Terminal showing 3 nginx pods running]*

> 💡 **Rithu's Tip:** *"See those random suffixes on pod names? That's the ReplicaSet adding a unique identifier. Each pod gets its own random hash!"*

---

### Step 2: Examine the ReplicaSet in Detail

```bash
kubectl describe replicaset nginx-rs
```

Key things to notice:
- **Replicas**: Shows desired vs current vs ready
- **Selector**: How the RS matches pods (by labels)
- **Events**: Shows scaling activities

> 💡 **Rithu's Tip:** *"The selector is CRITICAL. It's how the ReplicaSet knows which pods belong to it. If the selector doesn't match the pod labels, the ReplicaSet won't manage those pods!"*

---

### Step 3: Scale Replicas Up

```bash
# Scale using kubectl scale
kubectl scale replicaset nginx-rs --replicas=5
```

```bash
# Watch the new pods being created
kubectl get pods -l app=nginx -w
```

Wait for the output to show 5 running pods:
```
NAME             READY   STATUS    RESTARTS   AGE
nginx-rs-xxxxx   1/1     Running   0          1m
nginx-rs-yyyyy   1/1     Running   0          1m
nginx-rs-zzzzz   1/1     Running   0          1m
nginx-rs-aaaaa   1/1     Running   0          5s
nginx-rs-bbbbb   1/1     Running   0          5s
```

```bash
# Verify the ReplicaSet status
kubectl get replicaset nginx-rs
```

Expected output:
```
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   5         5         5       2m
```

📸 **Screenshot Placeholder:** *[Terminal showing 5 running pods]*

> 💡 **Rithu's Tip:** *"That was smooth! kubectl scale just told the ReplicaSet 'I want 5 pods' and it made it happen. Like ordering pizza — you just say how many slices you want!"*

🍕 **Rithu's Fun Fact:** *"Kubernetes was originally developed by Google and is based on their internal system called Borg. The name comes from Greek, meaning 'helmsman' or 'pilot'. So you're basically piloting a container ship now!"*

---

### Step 4: Scale Replicas Down

```bash
# Scale down to 2
kubectl scale replicaset nginx-rs --replicas=2
```

```bash
# Watch pods being terminated
kubectl get pods -l app=nginx -w
```

You'll see pods going from Running → Terminating → Gone:
```
NAME             READY   STATUS        AGE
nginx-rs-xxxxx   1/1     Running       3m
nginx-rs-yyyyy   1/1     Terminating   3m
nginx-rs-zzzzz   1/1     Terminating   3m
```

```bash
# Verify only 2 remain
kubectl get pods -l app=nginx
```

Expected output:
```
NAME             READY   STATUS    RESTARTS   AGE
nginx-rs-xxxxx   1/1     Running   0          3m
nginx-rs-yyyyy   1/1     Running   0          3m
```

---

### Step 5: Self-Healing Demo 🔥

This is where it gets exciting! Let's delete a pod and watch the ReplicaSet bring it back:

```bash
# Delete one pod manually
kubectl delete pod nginx-rs-xxxxx
```

**Immediately** watch the pods:
```bash
kubectl get pods -l app=nginx -w
```

You'll see:
```
NAME             READY   STATUS        AGE
nginx-rs-xxxxx   1/1     Terminating   4m
nginx-rs-yyyyy   1/1     Running       4m
nginx-rs-aaaaa   1/1     Running       2m
```

Then a few seconds later:
```
NAME             READY   STATUS    RESTARTS   AGE
nginx-rs-yyyyy   1/1     Running   0          4m
nginx-rs-aaaaa   1/1     Running   0          2m
nginx-rs-ccccc   1/1     Running   0          5s    ← NEW POD!
```

📸 **Screenshot Placeholder:** *[Terminal showing new pod created after deletion]*

> 💡 **Rithu's Tip:** *"The ReplicaSet controller is always watching. It constantly compares 'desired state' (3 replicas) with 'actual state' (2 running). When they don't match, it creates/deletes pods. This is the magic of Kubernetes — it's a reconciliation loop!"*

---

### Step 6: Edit ReplicaSet Inline

```bash
# Edit the ReplicaSet directly
kubectl edit replicaset nginx-rs
```

This opens your default editor. Change `replicas: 2` to `replicas: 4`, save and exit.

```bash
# Verify the change
kubectl get replicaset nginx-rs
```

Expected output:
```
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   4         4         4       5m
```

> 💡 **Rithu's Tip:** *"Editing inline is handy, but in production, you'd update the YAML file and re-apply it. That way, your changes are version-controlled!"*

---

### Step 7: Update the ReplicaSet Image

```bash
# Change the nginx image version
kubectl patch replicaset nginx-rs -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","image":"nginx:1.26"}]}}}}'
```

```bash
# Check the pods are being updated
kubectl get pods -l app=nginx -w
```

You'll see old pods terminating and new ones starting with the new image.

> 💡 **Rithu's Tip:** *"Wait — didn't we just manually update the image? This works, but there's a better way: Deployments! They handle rolling updates automatically and can rollback if something goes wrong. That's our next lab!"*

---

### Step 8: Understand ReplicaSet with YAML Breakdown

Let's dissect the YAML you created:

```yaml
apiVersion: apps/v1          # API group and version
kind: ReplicaSet             # Resource type
metadata:
  name: nginx-rs             # Name of the ReplicaSet
  labels:
    app: nginx               # Labels on the ReplicaSet itself
spec:
  replicas: 3                # How many pods to maintain
  selector:                  # HOW to find pods that belong to this RS
    matchLabels:
      app: nginx             # Must match pod labels
  template:                  # Pod template
    metadata:
      labels:
        app: nginx           # Labels on the pods (MUST match selector)
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
```

> 💡 **Rithu's Tip:** *"The selector.matchLabels and template.metadata.labels MUST match. If they don't, you'll get a confusing error like 'selector does not match template labels'. This catches more beginners than you'd think!"*

---

### Step 9: Check ReplicaSet Events

```bash
# See what's been happening
kubectl describe replicaset nginx-rs | grep -A 20 Events
```

Expected output:
```
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  10m    replicaset-controller  Created pod: nginx-rs-xxxxx
  Normal  SuccessfulCreate  10m    replicaset-controller  Created pod: nginx-rs-yyyyy
  Normal  SuccessfulCreate  10m    replicaset-controller  Created pod: nginx-rs-zzzzz
  Normal  SuccessfulCreate  8m     replicaset-controller  Created pod: nginx-rs-aaaaa
  Normal  SuccessfulCreate  8m     replicaset-controller  Created pod: nginx-rs-bbbbb
  Normal  SuccessfulDelete  6m     replicaset-controller  Deleted pod: nginx-rs-zzzzz
  Normal  SuccessfulDelete  6m     replicaset-controller  Deleted pod: nginx-rs-bbbbb
```

---

### Step 10: Clean Up

```bash
# Delete the ReplicaSet
kubectl delete replicaset nginx-rs
```

```bash
# Verify all pods are gone
kubectl get pods -l app=nginx
# Expected: No resources found in default namespace.
```

```bash
# Remove the YAML file
rm replicaset.yaml
```

---

## ✅ Verification

```bash
# 1. Create ReplicaSet
kubectl apply -f replicaset.yaml
kubectl get replicaset
# Expected: nginx-rs with 3/3/3 DESIRED/CURRENT/READY

# 2. Scale up
kubectl scale replicaset nginx-rs --replicas=4
kubectl get replicaset
# Expected: 4/4/4

# 3. Self-healing
kubectl delete pod -l app=nginx --field-selector=status.phase=Running | head -1
kubectl get pods -l app=nginx
# Expected: Back to 4 pods after a few seconds

# 4. Cleanup
kubectl delete replicaset nginx-rs
kubectl get pods
# Expected: No pods
```

---

## 🧹 Cleanup

```bash
kubectl delete replicaset nginx-rs
kubectl delete all --all
rm replicaset.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **ReplicaSet** | Maintains a stable set of replica pods running |
| **Replicas** | The desired number of identical pods |
| **Selector** | Label-based mechanism to identify which pods belong to the RS |
| **Self-healing** | RS automatically replaces failed/deleted pods |
| **Scaling** | Easily adjust the number of running pods |
| **Reconciliation** | The controller constantly compares desired vs actual state |

---

## 🚀 What's Next?

ReplicaSets are great, but they don't handle updates gracefully. Enter Deployments:

**[Lab 04: Deployments →](../04 - Deployments/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Self-healing pods? Scaling? You're becoming dangerous!  ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
