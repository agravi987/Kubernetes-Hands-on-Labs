# Lab 11 — Jobs & CronJobs

<div align="center">

![Lab-11](https://img.shields.io/badge/Lab-11-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-⭐⭐%20Medium-blue)
![Time](https://img.shields.io/badge/Time-25%20Minutes-orange)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-brightgreen)
![Service](https://img.shields.io/badge/Service-Jobs%20%2F%20CronJobs-326CE5)

```
╔══════════════════════════════════════════════════════════════╗
║  Lab 11 — Jobs & CronJobs                                    ║
║  "Run-to-Completion and Scheduled Tasks"                     ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

> *"Some things shouldn't run forever. A batch job processes your data, finishes, and goes to sleep forever. Like a good employee, unlike me — I never stop teaching!"* — **Rithu** 🧑‍🏫

---

## 🎯 Objective

By the end of this lab, you will:

- ✅ Create Jobs that run to completion
- ✅ Configure parallelism and completions
- ✅ Create CronJobs for scheduled tasks
- ✅ Understand Job backoff and retry policies

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-10
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
│                                                              │
│  JOB                                                          │
│  ┌────────────────────────────────────────────────────┐     │
│  │  Pod 1: ✅ Completed (exit 0)                       │     │
│  │  Pod 2: ✅ Completed (exit 0)                       │     │
│  │  Pod 3: 🔄 Running...                              │     │
│  │  Pod 4: ⏳ Pending                                  │     │
│  └────────────────────────────────────────────────────┘     │
│                                                              │
│  CRONJOB                                                    │
│  ┌────────────────────────────────────────────────────┐     │
│  │  Schedule: "*/5 * * * *" (every 5 minutes)          │     │
│  │                                                      │     │
│  │  12:00 → Creates Job → Pod runs → ✅ Done           │     │
│  │  12:05 → Creates Job → Pod runs → ✅ Done           │     │
│  │  12:10 → Creates Job → Pod runs → ✅ Done           │     │
│  │  12:15 → Creates Job → ...                          │     │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create a Simple Job

```bash
cat > simple-job.yaml << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox:latest
        command: ["sh", "-c", "echo 'Hello from Job!' && sleep 5 && echo 'Job complete!'"]
      restartPolicy: Never
  backoffLimit: 3
EOF
```

```bash
kubectl apply -f simple-job.yaml
kubectl get jobs
```

Expected output:
```
NAME        COMPLETIONS   DURATION   AGE
hello-job   0/1           10s        10s
```

```bash
# Watch the job complete
kubectl get jobs -w
```

Wait for COMPLETIONS to show `1/1`:
```
NAME        COMPLETIONS   DURATION   AGE
hello-job   0/1           15s        15s
hello-job   1/1           20s        20s
```

```bash
# Check the pod
kubectl get pods -l job-name=hello-job
```

Expected output:
```
NAME              READY   STATUS      RESTARTS   AGE
hello-job-xxxxx   0/1     Completed   0          1m
```

📸 **Screenshot Placeholder:** *[Terminal showing completed Job]*

> 💡 **Rithu's Tip:** *"Notice restartPolicy: Never? Jobs need either 'Never' or 'OnFailure'. 'Always' doesn't make sense for batch work!"*

---

### Step 2: View Job Logs

```bash
# Get logs from the completed job pod
kubectl logs job/hello-job
```

Expected output:
```
Hello from Job!
Job complete!
```

> 💡 **Rithu's Tip:** *"Even though the pod is 'Completed', you can still access its logs. The pod isn't deleted until the Job is cleaned up!"*

---

### Step 3: Parallel Jobs

Let's create a job that runs multiple pods in parallel:

```bash
cat > parallel-job.yaml << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job
spec:
  completions: 5
  parallelism: 3
  template:
    spec:
      containers:
      - name: worker
        image: busybox:latest
        command: ["sh", "-c", "echo 'Worker task $((RANDOM % 100)) completed' && sleep $((RANDOM % 5 + 1))"]
      restartPolicy: Never
  backoffLimit: 2
EOF
```

```bash
kubectl apply -f parallel-job.yaml
kubectl get jobs -w
```

You'll see:
```
NAME             COMPLETIONS   DURATION   AGE
parallel-job     0/5           10s        10s
parallel-job     2/5           15s        15s
parallel-job     3/5           20s        20s
parallel-job     5/5           25s        25s    ← All done!
```

📸 **Screenshot Placeholder:** *[Terminal showing parallel job completions]*

> 💡 **Rithu's Tip:** *"parallelism: 3 means up to 3 pods run simultaneously. completions: 5 means the job isn't done until 5 pods have successfully finished!"*

---

### Step 4: Indexed Jobs

```bash
cat > indexed-job.yaml << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: indexed-job
spec:
  completionMode: Indexed
  completions: 5
  parallelism: 5
  template:
    spec:
      containers:
      - name: worker
        image: busybox:latest
        command: ["sh", "-c", "echo 'Processing index $JOB_INDEX of $JOB_COMPLETIONS'"]
      restartPolicy: Never
EOF
```

```bash
kubectl apply -f indexed-job.yaml
kubectl get pods -l job-name=indexed-job -w
```

Wait for all to complete, then:

```bash
for pod in $(kubectl get pods -l job-name=indexed-job -o jsonpath='{.items[*].metadata.name}'); do
  echo "=== $pod ==="
  kubectl logs $pod
done
```

> 💡 **Rithu's Tip:** *"Indexed jobs give each pod an index number ($JOB_INDEX). Perfect for batch processing where you need to process items 1 through N!"*

---

### Step 5: Create a CronJob

```bash
cat > cronjob.yaml << 'EOF'
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cronjob
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:latest
            command: ["sh", "-c", "echo 'CronJob executed at $(date)'"]
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
EOF
```

```bash
kubectl apply -f cronjob.yaml
kubectl get cronjobs
```

Expected output:
```
NAME            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello-cronjob   */2 * * * *   False     0        <none>          10s
```

Wait 2 minutes, then:

```bash
kubectl get cronjobs
```

Expected output:
```
NAME            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello-cronjob   */2 * * * *   False     0        30s             2m
```

```bash
# See the jobs created by the CronJob
kubectl get jobs -l job-name=hello-cronjob
```

> 💡 **Rithu's Tip:** *"CronJobs create Jobs! Each schedule trigger creates a new Job, which creates pods. It's a chain: CronJob → Job → Pod."*

📸 **Screenshot Placeholder:** *[Terminal showing CronJob and its Jobs]*

---

### Step 6: CronJob Concurrency Policy

```bash
cat > cronjob-allow-concurrent.yaml << 'EOF'
apiVersion: batch/v1
kind: CronJob
metadata:
  name: concurrent-cronjob
spec:
  schedule: "*/1 * * * *"
  concurrencyPolicy: Allow
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: worker
            image: busybox:latest
            command: ["sh", "-c", "echo 'Starting long task...' && sleep 90 && echo 'Done!'"]
          restartPolicy: OnFailure
EOF
```

Concurrency policies:
- **Allow** (default): Multiple jobs can run simultaneously
- **Forbid**: Skip new job if previous is still running
- **Replace**: Cancel running job and start new one

> 💡 **Rithu's Tip:** *"Use 'Forbid' for tasks like database backups — you don't want two backup jobs running at the same time!"*

---

### Step 7: View CronJob Logs

```bash
# Get pods created by the cronjob
CRONJOB_PODS=$(kubectl get pods -l job-name=hello-cronjob -o jsonpath='{.items[-1].metadata.name}')
kubectl logs $CRONJOB_PODS
```

---

### Step 8: Clean Up

```bash
kubectl delete -f .
kubectl delete cronjob --all
kubectl delete job --all
rm *.yaml 2>/dev/null
```

---

## ✅ Verification

```bash
# 1. Simple job
kubectl apply -f simple-job.yaml
kubectl wait --for=condition=complete job/hello-job --timeout=60s
# Expected: Job completed

# 2. CronJob
kubectl apply -f cronjob.yaml
kubectl get cronjobs
# Expected: hello-cronjob with schedule

# 3. Cleanup
kubectl delete job --all
kubectl delete cronjob --all
```

---

## 🧹 Cleanup

```bash
kubectl delete job --all
kubectl delete cronjob --all
rm *.yaml 2>/dev/null
```

---

## 📝 What You Learned

| Concept | Description |
|---------|-------------|
| **Job** | Creates pods that run to completion |
| **CronJob** | Creates Jobs on a schedule |
| **completions** | Number of successful pod completions needed |
| **parallelism** | Number of pods running simultaneously |
| **backoffLimit** | Number of retries before marking job as failed |
| **concurrencyPolicy** | How CronJobs handle overlapping runs |
| **restartPolicy** | Never or OnFailure for Jobs |

---

## 🚀 What's Next?

Let's ensure pods run on every node:

**[Lab 12: DaemonSets →](../12 - DaemonSets/README.md)**

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  🎉 Batch jobs and scheduled tasks? You're automation       ║
║     is getting serious! ⏰                                  ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>
