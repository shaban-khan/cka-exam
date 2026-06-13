# This is arguably the **most important section of the CKA exam**.

Many candidates know how to create Pods, Deployments, and Services, but lose marks because they cannot **diagnose why something is broken**.

A useful mindset:

```text
CKA Troubleshooting = Follow the path

User
 ↓
Service
 ↓
Pod
 ↓
Container
 ↓
Node
 ↓
Cluster Components
```

When something fails, find where the chain breaks.

---

# 1. Troubleshoot Clusters and Nodes

# Theory

A Kubernetes cluster consists of:

```text
Control Plane
    ↓
Worker Nodes
    ↓
Pods
```

If a node is unhealthy:

```text
Pods may not start
Pods may be evicted
Scheduling may fail
```

---

# What Can Go Wrong?

### Node Not Ready

```text
STATUS = NotReady
```

Causes:

* kubelet stopped
* disk full
* memory pressure
* network issue

---

### Node Resource Exhaustion

```text
CPU 100%
Memory 100%
Disk full
```

Pods may remain:

```text
Pending
Evicted
CrashLoopBackOff
```

---

# First Commands

Check nodes:

```bash
kubectl get nodes
```

Example:

```text
NAME      STATUS
worker1   Ready
worker2   NotReady
```

---

Describe node:

```bash
kubectl describe node worker2
```

Look for:

```text
Conditions:
  MemoryPressure
  DiskPressure
  PIDPressure
```

---

# Example

Problem:

```text
Pod Pending
```

Check:

```bash
kubectl describe pod nginx
```

Output:

```text
0/2 nodes available:
Insufficient memory
```

Diagnosis:

```text
Node resource issue
```

---

# CKA Tip

When Pod is Pending:

```bash
kubectl describe pod <pod>
```

FIRST.

Never guess.

---

# Sample CKA Question

```text
Pod nginx is stuck in Pending state.
Determine and fix the issue.
```

Answer:

```bash
kubectl describe pod nginx
kubectl describe node worker1
```

---

# 2. Troubleshoot Cluster Components

# Theory

Control Plane components:

```text
kube-apiserver
kube-scheduler
kube-controller-manager
etcd
```

If any fail:

```text
Cluster partially broken
```

---

# API Server

Most important component.

Everything talks to:

```text
kube-apiserver
```

If down:

```text
kubectl commands fail
```

---

# Scheduler

Assigns Pods to nodes.

If down:

```text
Pods remain Pending
```

---

# Controller Manager

Maintains desired state.

If down:

```text
ReplicaSets stop creating Pods
```

---

# etcd

Stores cluster state.

If down:

```text
Cluster data unavailable
```

---

# Commands

Static Pods:

```bash
kubectl get pods -n kube-system
```

or

```bash
crictl ps
```

---

Logs:

```bash
kubectl logs -n kube-system kube-scheduler-<node>
```

or

```bash
crictl logs <container-id>
```

---

# Example

Problem:

```text
Pods stay Pending
```

Check:

```bash
kubectl get pods -n kube-system
```

Output:

```text
kube-scheduler CrashLoopBackOff
```

Root Cause:

```text
Scheduler failure
```

---

# CKA Tip

Remember:

```bash
kubectl get pods -n kube-system
```

This command solves many troubleshooting questions.

---

# Sample Question

```text
A control plane component is unhealthy.
Identify the issue.
```

Check:

```bash
kubectl get pods -n kube-system
kubectl logs
```

---

# 3. Monitor Cluster and Application Resource Usage

# Theory

Applications consume:

```text
CPU
Memory
Storage
```

When resources run out:

```text
Slow performance
OOMKilled
Pending Pods
```

---

# Metrics Server

Provides:

```text
CPU usage
Memory usage
```

---

# Commands

Node usage:

```bash
kubectl top node
```

Output:

```text
NAME      CPU   MEMORY
worker1   45%   60%
```

---

Pod usage:

```bash
kubectl top pod
```

---

Namespace:

```bash
kubectl top pod -n app1
```

---

# Example

Problem:

```text
Application restarting
```

Check:

```bash
kubectl describe pod
```

Output:

```text
OOMKilled
```

Meaning:

```text
Memory limit exceeded
```

---

# CKA Tip

Always check:

```bash
kubectl top pod
kubectl top node
```

when performance issues appear.

---

# Sample Question

```text
Application keeps restarting.
Find the reason.
```

Answer:

```bash
kubectl describe pod
kubectl top pod
```

Look for:

```text
OOMKilled
```

---

# 4. Manage and Evaluate Container Output Streams

# Theory

Containers write logs to:

```text
stdout
stderr
```

Kubernetes collects these streams.

---

# Why Important?

Most application failures are visible in logs.

Example:

```text
Database connection failed
```

appears in logs.

---

# Commands

Current logs:

```bash
kubectl logs nginx
```

---

Multi-container:

```bash
kubectl logs nginx -c app
```

---

Follow logs:

```bash
kubectl logs -f nginx
```

---

Previous container:

```bash
kubectl logs nginx --previous
```

Very important.

---

# Example

Problem:

```text
CrashLoopBackOff
```

Check:

```bash
kubectl logs pod-name --previous
```

Output:

```text
Missing environment variable
```

Root cause found.

---

# CKA Tip

For CrashLoopBackOff:

```bash
kubectl logs --previous
```

Almost always.

---

# Sample Question

```text
Pod is crashing repeatedly.
Identify the cause.
```

Answer:

```bash
kubectl logs pod --previous
```

---

# 5. Troubleshoot Services and Networking

This is heavily tested.

---

# Theory

Traffic path:

```text
Client
 ↓
Service
 ↓
Endpoints
 ↓
Pod
```

If broken:

```text
Application inaccessible
```

---

# Step 1: Check Pod

```bash
kubectl get pods
```

Must be:

```text
Running
Ready
```

---

# Step 2: Check Service

```bash
kubectl get svc
```

---

# Step 3: Check Endpoints

Most important command:

```bash
kubectl get endpoints
```

Example:

```text
nginx-service
  10.244.1.5:80
```

No endpoints:

```text
<none>
```

means Service selector issue.

---

# Example

Service:

```yaml
selector:
  app: nginx
```

Pod:

```yaml
labels:
  app: web
```

Result:

```text
No endpoints
```

Fix:

```text
Match labels
```

---

# DNS Troubleshooting

Check:

```bash
kubectl run test --image=busybox -it --rm
```

Inside:

```bash
nslookup kubernetes.default
```

Should resolve.

---

# Service Connectivity

Test:

```bash
curl http://service-name
```

or

```bash
wget -qO- service-name
```

---

# CKA Tip

Always follow:

```text
Pod
 ↓
Service
 ↓
Endpoints
 ↓
DNS
```

Never jump randomly.

---

# Sample Question

```text
Service nginx-svc is not reachable.
Find and fix the issue.
```

Steps:

```bash
kubectl get pod
kubectl get svc
kubectl get endpoints
kubectl describe svc nginx-svc
```

---

# The Most Important Troubleshooting Commands for CKA

### Pods

```bash
kubectl get pod
kubectl describe pod
kubectl logs
kubectl logs --previous
```

---

### Nodes

```bash
kubectl get nodes
kubectl describe node
kubectl top node
```

---

### Resources

```bash
kubectl top pod
kubectl top node
```

---

### Services

```bash
kubectl get svc
kubectl describe svc
kubectl get endpoints
```

---

### Control Plane

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system
```

---

# Golden CKA Troubleshooting Flow

Whenever something is broken:

```text
1. kubectl get
2. kubectl describe
3. kubectl logs
4. kubectl top
5. Check selectors/endpoints
6. Check kube-system
```

If you consistently follow that sequence instead of guessing, you'll solve the majority of troubleshooting tasks that appear in the CKA exam.
