That's fair. For CKA preparation, each topic should be learned in this order:

```text
1. What is it? (Theory)
2. Why do we need it?
3. How does it work internally?
4. Simple example
5. CKA commands
6. Common exam scenarios
```

---

# 1. Understand Application Deployments and Perform Rolling Updates/Rollbacks

## Theory

A Deployment is a Kubernetes controller that manages the lifecycle of applications.

Before Kubernetes:

```text
Application
   ↓
Single Process
```

If the process crashed:

```text
Application Down
```

In Kubernetes:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

The Deployment continuously ensures that the desired number of Pods are running.

---

## Why Do We Need Deployments?

Problems in real-world applications:

### Problem 1: Pod Failure

```text
Pod dies
```

Need:

```text
Automatic replacement
```

### Problem 2: Application Upgrade

```text
Version 1 → Version 2
```

Need:

```text
No downtime
```

### Problem 3: Bad Upgrade

```text
Version 2 broken
```

Need:

```text
Rollback
```

Deployments solve all three.

---

## How It Works Internally

```text
Deployment
      ↓
Creates ReplicaSet
      ↓
ReplicaSet creates Pods
```

Example:

```text
replicas: 3
```

Desired state:

```text
3 Pods
```

Actual state:

```text
2 Pods
```

ReplicaSet creates:

```text
1 new Pod
```

---

## Rolling Update Theory

Old version:

```text
nginx:1.25
```

New version:

```text
nginx:1.26
```

Instead of:

```text
Stop all old Pods
Start all new Pods
```

Kubernetes does:

```text
Create new Pod
Wait Ready
Delete old Pod
```

one at a time.

This is called:

```text
Rolling Update
```

---

## Rollback Theory

Deployment stores revision history.

Example:

```text
Revision 1 → nginx:1.25
Revision 2 → nginx:1.26
```

If Revision 2 fails:

```text
Rollback
```

Kubernetes restores:

```text
Revision 1
```

---

## Simple Example

```bash
kubectl create deployment nginx --image=nginx
kubectl set image deployment/nginx nginx=nginx:1.26
kubectl rollout undo deployment/nginx
```

---

# 2. Use ConfigMaps and Secrets to Configure Applications

## Theory

Applications require configuration.

Examples:

```text
Database host
Application color
Environment name
API key
Password
```

You should not hardcode configuration inside container images.

Bad:

```text
Docker Image
 └── DB_HOST=prod-db
```

Need new image for every environment.

---

## ConfigMap Theory

A ConfigMap stores non-sensitive configuration separately from the application.

Examples:

```text
APP_COLOR=blue
LOG_LEVEL=debug
```

Benefits:

```text
Change configuration
Without rebuilding image
```

---

## Secret Theory

A Secret stores sensitive information.

Examples:

```text
Passwords
API keys
Certificates
Tokens
```

Benefits:

```text
Separate credentials from application code
```

---

## How It Works

```text
ConfigMap/Secret
        ↓
Environment Variable
        OR
Mounted File
        ↓
Application
```

---

## Simple Example

ConfigMap:

```bash
kubectl create configmap app-config \
--from-literal=color=blue
```

Secret:

```bash
kubectl create secret generic db-secret \
--from-literal=password=Pass123
```

---

# 3. Configure Workload Autoscaling

## Theory

Traffic is never constant.

Example:

```text
Morning
100 users

Evening
10,000 users
```

If application always runs:

```text
2 Pods
```

Then:

```text
High CPU
Slow response
```

If application always runs:

```text
50 Pods
```

Then:

```text
Resource waste
```

Need automatic scaling.

---

## Types of Scaling

### Horizontal Scaling

```text
2 Pods → 5 Pods → 10 Pods
```

Add more Pods.

Implemented using:

```text
HPA
(Horizontal Pod Autoscaler)
```

---

### Vertical Scaling

```text
CPU 100m → CPU 1000m
```

Increase Pod resources.

Not commonly tested in CKA.

---

## HPA Theory

HPA continuously monitors:

```text
CPU
Memory
Custom Metrics
```

When utilization increases:

```text
More Pods
```

When utilization decreases:

```text
Fewer Pods
```

---

## How It Works

```text
Metrics Server
      ↓
HPA
      ↓
Deployment
      ↓
Pods
```

---

## Simple Example

```bash
kubectl autoscale deployment web \
--cpu-percent=50 \
--min=2 \
--max=10
```

---

# 4. Understand Primitives Used to Create Robust, Self-Healing Deployments

## Theory

Self-healing means:

```text
System automatically recovers from failures.
```

Core Kubernetes principle:

```text
Desired State
```

You declare:

```text
3 Pods
```

Kubernetes ensures:

```text
3 Pods
```

always exist.

---

## ReplicaSet

Purpose:

```text
Maintain desired number of Pods.
```

Example:

```text
Desired = 3
Actual = 2
```

ReplicaSet creates:

```text
1 new Pod
```

---

## Liveness Probe

Question:

```text
Is the application alive?
```

If answer is:

```text
No
```

Kubernetes:

```text
Restart container
```

---

## Readiness Probe

Question:

```text
Can application receive traffic?
```

If answer is:

```text
No
```

Service stops sending traffic.

---

## Restart Policy

Determines:

```text
When container exits
What should happen?
```

Common:

```text
Always
```

---

## Benefits

```text
Automatic recovery
Reduced downtime
Higher availability
```

---

# 5. Configure Pod Admission and Scheduling

## Theory

Pods do not select nodes.

The Kubernetes Scheduler selects nodes.

Question:

```text
Where should this Pod run?
```

Scheduler evaluates:

```text
Resources
Labels
Affinity Rules
Taints
Policies
```

---

## Requests and Limits

### Requests

Minimum resources required.

Example:

```yaml
requests:
  cpu: 100m
  memory: 128Mi
```

Think:

```text
Reservation
```

---

### Limits

Maximum resources allowed.

```yaml
limits:
  cpu: 500m
  memory: 256Mi
```

Think:

```text
Speed Limit
```

---

## Node Selector

Simple scheduling rule.

```text
Run only on nodes
With specific labels
```

Example:

```yaml
nodeSelector:
  disktype: ssd
```

---

## Node Affinity

Advanced node selection.

Supports:

```text
Required
Preferred
Complex rules
```

Example:

```text
Run on SSD nodes
Prefer east region
```

---

## Taints

Applied to Nodes.

Meaning:

```text
Keep Pods Away
```

Example:

```text
Database Node
GPU Node
```

---

## Tolerations

Applied to Pods.

Meaning:

```text
This Pod is allowed
On tainted node
```

---

## Pod Affinity

Purpose:

```text
Place Pods Together
```

Example:

```text
Web Pod
Cache Pod
```

Benefits:

```text
Low latency
```

---

## Pod Anti-Affinity

Purpose:

```text
Spread Pods Apart
```

Example:

```text
Frontend Replica 1 → Node1
Frontend Replica 2 → Node2
Frontend Replica 3 → Node3
```

Benefits:

```text
High Availability
Fault Tolerance
```

---

# CKA Exam Focus Areas

If you have only one day before the exam, memorize:

### Deployments

```bash
kubectl create deployment
kubectl set image
kubectl rollout status
kubectl rollout undo
```

### ConfigMaps & Secrets

```bash
kubectl create configmap
kubectl create secret
```

### Autoscaling

```bash
kubectl autoscale deployment
kubectl get hpa
```

### Self-Healing

```text
ReplicaSet
Liveness Probe
Readiness Probe
Restart Policy
```

### Scheduling

```text
Requests
Limits
NodeSelector
NodeAffinity
Taints
Tolerations
PodAffinity
PodAntiAffinity
```

These are the concepts behind almost every Workloads & Scheduling question you'll see in the CKA exam.
