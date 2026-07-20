# Lab 21 — Monitoring with Prometheus & Grafana

<div align="center">

![Lab-21](https://img.shields.io/badge/Lab-21-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐⭐⭐%20Hard-red)
![Time](https://img.shields.io/badge/Time-45%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Prometheus%20%2F%20Grafana-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 21 — Monitoring with Prometheus & Grafana                ║
║  "See Everything Happening in Your Cluster"                  ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"You can't improve what you can't measure. Prometheus collects the numbers, Grafana makes them pretty, and together they give you superpowers!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Deploy kube-prometheus-stack via Helm
- ✅ Access Grafana dashboards
- ✅ View Prometheus metrics
- ✅ Understand basic alerting

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-20
- [ ] Minikube running with 4GB+ RAM
- [ ] Helm installed

---

## 💰 Cost Warning

```
💵 COST: $0.00 — Using Minikube (local)
⚠️  Needs at least 4GB RAM for the monitoring stack!
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   MONITORING STACK                               │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Prometheus                               │  │
│  │  ├── Scrapes metrics from all pods/nodes                 │  │
│  │  ├── Stores time-series data                             │  │
│  │  └── Evaluates alerting rules                             │  │
│  └────────────────────────┬─────────────────────────────────┘  │
│                           │                                      │
│  ┌────────────────────────▼─────────────────────────────────┐  │
│  │                    Grafana                                 │  │
│  │  ├── Beautiful dashboards                                 │  │
│  │  ├── Query Prometheus                                     │  │
│  │  └── Alert on metrics                                     │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Alertmanager                            │  │
│  │  ├── Receives alerts from Prometheus                      │  │
│  │  ├── Routes to email/Slack/PagerDuty                      │  │
│  │  └── Deduplicates and groups alerts                       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  METRICS FLOW:                                                   │
│  Pods/Nodes → Prometheus → Grafana → YOU! 📊                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Add Helm Repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

### Step 2: Install kube-prometheus-stack

```bash
# Create monitoring namespace
kubectl create namespace monitoring

# Install the stack
helm install kube-prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.adminPassword=admin \
  --set grafana.service.type=NodePort \
  --set grafana.service.nodePort=30000 \
  --set prometheus.service.type=NodePort \
  --set prometheus.service.nodePort=30001
```

This takes a few minutes. Wait for all pods to be running:

```bash
kubectl get pods -n monitoring -w
```

Expected output:
```
NAME                                                   READY   STATUS    AGE
alertmanager-kube-prometheus-kube-prometheus-alert...  2/2     Running   2m
kube-prometheus-grafana-xxxxxxxxx-xxxxx                3/3     Running   3m
kube-prometheus-kube-prometheus-operator-xxxxx         1/1     Running   3m
kube-prometheus-kube-prometheus-prometheus-node...     2/2     Running   2m
prometheus-kube-prometheus-kube-prometheus-prom...     2/2     Running   2m
```

📸 **Screenshot Placeholder:** *[Terminal showing monitoring pods running]*

---

### Step 3: Access Grafana Dashboard

```bash
# Get the Grafana URL
minikube service kube-prometheus-grafana -n monitoring --url
```

Open the URL in your browser:
- **Username:** admin
- **Password:** admin

You'll see Grafana with pre-built dashboards for:
- Kubernetes cluster monitoring
- Node exporter metrics
- Pod/container metrics

📸 **Screenshot Placeholder:** *[Browser showing Grafana dashboard]*

> 💡 **Rithu's Tip:** *"Grafana comes with amazing pre-built dashboards! Click 'Dashboards' → 'Browse' to see all available dashboards!"*

---

### Step 4: Access Prometheus

```bash
# Get the Prometheus URL
minikube service kube-prometheus-kube-prometheus-prometheus -n monitoring --url
```

Open the URL and try these PromQL queries:

```promql
# CPU usage per pod
rate(container_cpu_usage_seconds_total{namespace="default"}[5m])

# Memory usage per pod
container_memory_working_set_bytes{namespace="default"}

# Pod count
count(kube_pod_info{namespace="default"})
```

📸 **Screenshot Placeholder:** *[Browser showing Prometheus UI with query results]*

---

### Step 5: Explore Key Metrics

In Prometheus, try these useful queries:

```promql
# Nodes CPU usage
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Pod restart count
kube_pod_container_status_restarts_total

# PVC usage
kubelet_volume_stats_used_bytes / kubelet_volume_stats_capacity_bytes * 100
```

> 💡 **Rithu's Tip:** *"PromQL is Prometheus's query language. Learn a few basic functions and you can build powerful dashboards!"*

---

### Step 6: View Alertmanager

```bash
# Get Alertmanager URL
minikube service alertmanager-kube-prometheus-kube-prometheus-alertmanager -n monitoring --url
```

Alertmanager shows active alerts and their status.

---

### Step 7: Clean Up

```bash
# Uninstall the monitoring stack
helm uninstall kube-prometheus -n monitoring
kubectl delete namespace monitoring
```

---

## ✅ Verification

```bash
# 1. Helm chart installed
helm list -n monitoring
# Expected: kube-prometheus deployed

# 2. Grafana accessible
minikube service kube-prometheus-grafana -n monitoring --url
# Expected: URL to access Grafana

# 3. Prometheus working
minikube service kube-prometheus-kube-prometheus-prometheus -n monitoring --url
# Expected: URL to access Prometheus

# 4. Cleanup
helm uninstall kube-prometheus -n monitoring
kubectl delete namespace monitoring
```

---

## 🧹 Cleanup

```bash
helm uninstall kube-prometheus -n monitoring 2>/dev/null
kubectl delete namespace monitoring 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Prometheus** | Time-series metrics collection and storage |
| **Grafana** | Beautiful visualization dashboards |
| **Alertmanager** | Alert routing and notification |
| **kube-prometheus-stack** | All-in-one monitoring Helm chart |
| **PromQL** | Prometheus query language |
| **ServiceMonitor** | Defines how Prometheus scrapes targets |

---

## 🚀 What's Next?

Let's add centralized logging:

**[Lab 22: Logging with EFK Stack →](../22 - Logging with EFK Stack/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Your cluster now has full visibility! Metrics,          ║
║     dashboards, and alerts — oh my! 📊                      ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
