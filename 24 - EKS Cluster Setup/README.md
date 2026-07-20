# Lab 24 — EKS Cluster Setup

<div align="center">

![Lab-24](https://img.shields.io/badge/Lab-24-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐⭐%20Hard-red)
![Time](https://img.shields.io/badge/Time-60%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Pay%20as%20you%20go%20⚠️-yellow)
![Service](https://img.shields.io/badge/Service-EKS-FF9900)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 24 — EKS Cluster Setup                                  ║
║  "Production-Grade Kubernetes on AWS"                        ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Minikube is your training wheels. EKS is the real deal — a managed Kubernetes cluster in the cloud, with AWS handling the control plane so you can focus on your apps!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Create an EKS cluster using `eksctl`
- ✅ Understand managed node groups
- ✅ Configure aws-auth ConfigMap
- ✅ Deploy apps to EKS
- ✅ Clean up to avoid charges

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-23
- [ ] AWS account with programmatic access
- [ ] AWS CLI configured
- [ ] eksctl installed
- [ ] kubectl installed

---

## 💰 Cost Warning

```
⚠️  COST WARNING: This lab creates AWS resources that incur charges!
    ├── EKS Control Plane: ~$0.10/hour ($72/month)
    ├── EC2 Nodes: ~$0.05-0.20/hour (depends on instance type)
    └── TOTAL: ~$0.15-0.30/hour = ~$2-5 for this lab

    🔴 IMPORTANT: Clean up ALL resources when done!
    🔴 Use AWS Free Tier if eligible
    🔴 Set up billing alerts!
```

> *"This is the only lab that costs money. If you're on the free tier, you might get away with $0. But always clean up!"* — **Rithu** 🧑‍🏫

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    AWS EKS ARCHITECTURE                           │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    AWS Cloud                               │   │
│  │                                                            │   │
│  │  ┌────────────────────────────────────────────────────┐   │   │
│  │  │              VPC (Virtual Private Cloud)            │   │   │
│  │  │                                                      │   │   │
│  │  │  ┌──────────────────────────────────────────────┐  │   │   │
│  │  │  │           EKS Control Plane (Managed)         │  │   │   │
│  │  │  │  ├── API Server (multi-AZ)                    │  │   │   │
│  │  │  │  ├── etcd (encrypted)                        │  │   │   │
│  │  │  │  ├── Scheduler                               │  │   │   │
│  │  │  │  └── Controller Manager                      │  │   │   │
│  │  │  └──────────────────────────────────────────────┘  │   │   │
│  │  │                                                      │   │   │
│  │  │  ┌──────────────────────────────────────────────┐  │   │   │
│  │  │  │           Managed Node Group                  │  │   │   │
│  │  │  │                                              │  │   │   │
│  │  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐     │  │   │   │
│  │  │  │  │  Node 1  │  │  Node 2  │  │  Node 3  │     │  │   │   │
│  │  │  │  │ (EC2)    │  │ (EC2)    │  │ (EC2)    │     │  │   │   │
│  │  │  │  │ t3.medium│  │ t3.medium│  │ t3.medium│     │  │   │   │
│  │  │  │  └─────────┘  └─────────┘  └─────────┘     │  │   │   │
│  │  │  └──────────────────────────────────────────────┘  │   │   │
│  │  └────────────────────────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  MANAGED BY:                                                     │
│  ├── AWS manages Control Plane (you don't see it)              │
│  ├── You manage Worker Nodes (via Node Groups)                  │
│  └── eksctl simplifies everything                               │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Verify AWS Configuration

```bash
# Check AWS CLI is configured
aws sts get-caller-identity
```

Expected output:
```
{
    "UserId": "AIDAXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-username"
}
```

```bash
# Check default region
aws configure get region
```

---

### Step 2: Create EKS Cluster

```bash
# Create a cluster (takes 15-20 minutes!)
eksctl create cluster \
  --name my-first-eks-cluster \
  --region us-west-2 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```

> 💡 **Rithu's Tip:** *"This command creates: VPC, subnets, security groups, EKS control plane, and 2 worker nodes. It's a LOT of infrastructure — and it takes about 15-20 minutes!"*

📸 **Screenshot Placeholder:** *[Terminal showing eksctl creating cluster]*

---

### Step 3: Verify Cluster

```bash
# kubectl is automatically configured
kubectl cluster-info
kubectl get nodes
```

Expected output:
```
NAME                           STATUS   ROLES    AGE   VERSION
ip-172-31-xx-xx.ec2.internal   Ready    <none>   5m    v1.28.x
ip-172-31-yy-yy.ec2.internal   Ready    <none>   5m    v1.28.x
```

> 💡 **Rithu's Tip:** *"Your nodes are now EC2 instances running in AWS! They're managed by EKS and connected to the managed control plane."*

📸 **Screenshot Placeholder:** *[Terminal showing EKS nodes in Ready state]*

---

### Step 4: Deploy an Application

```bash
cat > eks-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-eks
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-eks
  template:
    metadata:
      labels:
        app: nginx-eks
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-eks-svc
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
  type: LoadBalancer
  selector:
    app: nginx-eks
  ports:
  - port: 80
    targetPort: 80
EOF
```

```bash
kubectl apply -f eks-deployment.yaml
kubectl get pods -o wide
kubectl get svc nginx-eks-svc
```

Wait for the EXTERNAL-IP to appear:
```
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP                              PORT(S)        AGE
nginx-eks-svc  LoadBalancer   10.100.xx.xxx   aaaaaaa-xxxx.us-west-2.elb.amazonaws.com 80:3XXXX/TCP   2m
```

Open the EXTERNAL-IP in your browser — nginx welcome page!

📸 **Screenshot Placeholder:** *[Terminal showing nginx running on EKS with external Load Balancer]*

> 💡 **Rithu's Tip:** *"That ELB (Elastic Load Balancer) was automatically created by AWS when you used type: LoadBalancer. This is the cloud magic!"*

---

### Step 5: Check aws-auth ConfigMap

```bash
# View the aws-auth ConfigMap
kubectl get configmap aws-auth -n kube-system -o yaml
```

This ConfigMap maps AWS IAM roles to Kubernetes users/groups. It's how eksctl configured node access.

> 💡 **Rithu's Tip:** *"aws-auth is the bridge between AWS IAM and Kubernetes RBAC. It tells Kubernetes 'trust this IAM role' or 'this IAM user is this K8s user'."*

---

### Step 6: Monitor Cluster

```bash
# Check cluster autoscaler (if enabled)
kubectl get deployment -n kube-system | grep autoscaler

# Check system pods
kubectl get pods -n kube-system
```

---

### Step 7: CLEAN UP (CRITICAL!)

```bash
# Delete the cluster (takes 10-15 minutes)
eksctl delete cluster --name my-first-eks-cluster --region us-west-2
```

> ⚠️ **WARNING:** Do NOT skip this step! The cluster costs money every hour it's running!

```bash
# Verify cleanup
kubectl cluster-info 2>&1
# Expected: connection refused (cluster is gone)
```

```bash
# Also delete any ELBs that might have been created
aws elbv2 describe-load-balancers --query "LoadBalancers[?contains(LoadBalancerName, 'k8s')].{Name:LoadBalancerName,ARN:LoadBalancerArn}"
```

> 💡 **Rithu's Tip:** *"If you see orphaned ELBs, delete them manually. They cost money too! The eksctl delete should handle most of this, but always double-check!"*

---

## ✅ Verification

```bash
# 1. Cluster created
eksctl get cluster --name my-first-eks-cluster
# Expected: Cluster details

# 2. Nodes ready
kubectl get nodes
# Expected: 2 nodes in Ready state

# 3. App deployed
kubectl get svc nginx-eks-svc
# Expected: EXTERNAL-IP assigned

# 4. CLEANUP (most important!)
eksctl delete cluster --name my-first-eks-cluster --region us-west-2
```

---

## 🧹 Cleanup

```bash
# This is CRITICAL — don't leave AWS resources running!
eksctl delete cluster --name my-first-eks-cluster --region us-west-2
rm eks-deployment.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **EKS** | Managed Kubernetes on AWS |
| **eksctl** | CLI for creating/managing EKS clusters |
| **Managed Node Groups** | AWS-managed worker nodes |
| **aws-auth ConfigMap** | Maps IAM to Kubernetes RBAC |
| **LoadBalancer Service** | Creates AWS ELB automatically |
| **VPC** | Virtual networking for your cluster |

---

## 🚀 What's Next?

Let's put it all together in the capstone project:

**[Lab 25: Capstone — Microservices on Kubernetes →](../25 - Capstone Microservices/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 You just deployed a production cluster on AWS!          ║
║     That's the real deal! 🌩️                                ║
║                                                              ║
║  🔴 Did you clean up? Double-check! 💰                     ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
