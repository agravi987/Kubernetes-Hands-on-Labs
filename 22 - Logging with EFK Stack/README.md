# Lab 22 — Logging with EFK Stack

<div align="center">

![Lab-22](https://img.shields.io/badge/Lab-22-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐⭐%20Hard-red)
![Time](https://img.shields.io/badge/Time-45%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Elasticsearch%20%2F%20Fluentd%20%2F%20Kibana-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 22 — Logging with EFK Stack                             ║
║  "Centralized Logging for All Your Pods"                     ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"kubectl logs only shows one pod at a time. In production with 100 pods, you need centralized logging. EFK is the classic solution: Elasticsearch stores it, Fluentd collects it, Kibana shows it!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Understand the EFK stack architecture
- ✅ Deploy Elasticsearch
- ✅ Deploy Fluentd as a DaemonSet
- ✅ Deploy Kibana for log visualization
- ✅ Search logs across all pods

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-21
- [ ] Minikube running with 4GB+ RAM

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
⚠️  Needs at least 4GB RAM for EFK stack!
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    EFK STACK ARCHITECTURE                        │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Fluentd (DaemonSet)                                      │  │
│  │  ├── Runs on every node                                   │  │
│  │  ├── Collects container logs                              │  │
│  │  └── Forwards to Elasticsearch                            │  │
│  └────────────────────┬─────────────────────────────────────┘  │
│                       │                                          │
│                       ▼                                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Elasticsearch                                             │  │
│  │  ├── Stores and indexes logs                              │  │
│  │  ├── Full-text search                                     │  │
│  │  └── RESTful API                                          │  │
│  └────────────────────┬─────────────────────────────────────┘  │
│                       │                                          │
│                       ▼                                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Kibana                                                    │  │
│  │  ├── Web UI for log visualization                         │  │
│  │  ├── Search and filter logs                               │  │
│  │  └── Create dashboards                                    │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  LOG FLOW:                                                       │
│  Container stdout → Fluentd → Elasticsearch → Kibana → YOU! 📋│
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create Namespace

```bash
kubectl create namespace logging
```

---

### Step 2: Deploy Elasticsearch

```bash
cat > elasticsearch.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: elasticsearch
  namespace: logging
  labels:
    app: elasticsearch
spec:
  replicas: 1
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
        env:
        - name: discovery.type
          value: single-node
        - name: xpack.security.enabled
          value: "false"
        - name: ES_JAVA_OPTS
          value: "-Xms512m -Xmx512m"
        ports:
        - containerPort: 9200
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "1000m"
            memory: "2Gi"
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
  namespace: logging
spec:
  selector:
    app: elasticsearch
  ports:
  - port: 9200
    targetPort: 9200
EOF
```

```bash
kubectl apply -f elasticsearch.yaml
kubectl get pods -n logging -w
```

Wait for Elasticsearch to be Running. This takes a few minutes.

📸 **Screenshot Placeholder:** *[Terminal showing Elasticsearch pod running]*

---

### Step 3: Deploy Fluentd (DaemonSet)

```bash
cat > fluentd-configmap.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
  namespace: logging
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag kubernetes.*
      read_from_head true
      <parse>
        @type json
        time_format %Y-%m-%dT%H:%M:%S.%NZ
      </parse>
    </source>

    <filter kubernetes.**>
      @type kubernetes_metadata
    </filter>

    <match **>
      @type elasticsearch
      host elasticsearch
      port 9200
      logstash_format true
      logstash_prefix kubernetes
      <buffer>
        flush_interval 5s
      </buffer>
    </match>
EOF
```

```bash
cat > fluentd-daemonset.yaml << 'EOF'
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: logging
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      serviceAccountName: fluentd
      serviceAccount: fluentd
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.16-debian-elasticsearch8-1
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch"
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: containers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: config
          mountPath: /fluentd/etc/
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: containers
        hostPath:
          path: /var/lib/docker/containers
      - name: config
        configMap:
          name: fluentd-config
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: logging
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fluentd
rules:
- apiGroups: [""]
  resources: ["pods", "namespaces"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: fluentd
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: logging
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
EOF
```

```bash
kubectl apply -f fluentd-configmap.yaml
kubectl apply -f fluentd-daemonset.yaml
kubectl get pods -n logging -w
```

---

### Step 4: Deploy Kibana

```bash
cat > kibana.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: logging
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: docker.elastic.co/kibana/kibana:8.10.0
        env:
        - name: ELASTICSEARCH_HOSTS
          value: "http://elasticsearch:9200"
        ports:
        - containerPort: 5601
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "1000m"
            memory: "2Gi"
---
apiVersion: v1
kind: Service
metadata:
  name: kibana
  namespace: logging
spec:
  type: NodePort
  selector:
    app: kibana
  ports:
  - port: 5601
    targetPort: 5601
    nodePort: 30561
EOF
```

```bash
kubectl apply -f kibana.yaml
kubectl get pods -n logging -w
```

Wait for all pods to be Running.

---

### Step 5: Generate Some Logs

```bash
# Generate logs by creating a busybox pod
kubectl run log-generator --image=busybox:latest -- sh -c "for i in \$(seq 1 100); do echo \"Log message \$i from log-generator\"; sleep 1; done"
```

---

### Step 6: Access Kibana

```bash
minikube service kibana -n logging --url
```

Open in browser:
1. Click "Discover" in the left menu
2. Create an index pattern: `kubernetes-*`
3. Set `@timestamp` as the time field
4. Click "Discover" again — you should see logs!

📸 **Screenshot Placeholder:** *[Browser showing Kibana with logs]*

> 💡 **Rithu's Tip:** *"Kibana takes a minute to index logs. If you don't see anything, wait a bit and refresh!"*

---

### Step 7: Clean Up

```bash
kubectl delete namespace logging
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. Elasticsearch running
kubectl get pods -n logging -l app=elasticsearch

# 2. Fluentd running on nodes
kubectl get pods -n logging -l app=fluentd

# 3. Kibana accessible
minikube service kibana -n logging --url

# 4. Cleanup
kubectl delete namespace logging
```

---

## 🧹 Cleanup

```bash
kubectl delete namespace logging
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Elasticsearch** | Distributed search and analytics engine for logs |
| **Fluentd** | Log collector that runs as DaemonSet |
| **Kibana** | Web UI for log visualization and search |
| **Log Aggregation** | Collecting logs from all pods into one place |
| **DaemonSet Logging** | Fluentd runs on every node to collect local logs |

---

## 🚀 What's Next?

Let's learn about backups:

**[Lab 23: Backup with Velero →](../23 - Backup with Velero/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Centralized logging is now your superpower!              ║
║     No more SSH-ing into pods to check logs! 📋            ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
