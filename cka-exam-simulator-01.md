# CKA exam simulator 01
That's actually the missing piece for CKA preparation. Memorizing commands is not enough—you need to understand **why** you're running them so you can handle variations in the exam.

---

# Q1. Kubeconfig Extraction

### What is being tested?

Understanding the structure of a kubeconfig file:

```yaml
clusters:
users:
contexts:
current-context:
```

A context connects:

```text
User + Cluster + Namespace
```

Example:

```text
admin@prod
developer@staging
```

### How to think during exam

Question asks:

> Write all contexts

→ Look under:

```yaml
contexts:
```

Question asks:

> Current context

→ Look under:

```yaml
current-context:
```

Question asks:

> Extract certificate

→ Look under:

```yaml
users:
```

Many kubeconfigs store certificates as:

```yaml
client-certificate-data:
```

which is Base64 encoded.

Hence:

```bash
base64 -d
```

is needed.

---

# Q2. Cert Manager

### What is being tested?

Knowledge of:

* Helm
* CRDs
* Certificate automation

Cert Manager installs many Custom Resource Definitions:

```text
Certificate
Issuer
ClusterIssuer
CertificateRequest
```

Without CRDs:

```bash
kubectl apply -f issuer.yaml
```

fails.

Therefore:

```bash
--set crds.enabled=true
```

is critical.

### How to think

Whenever exam says:

```text
Install cert-manager
```

Immediately think:

```text
Namespace
Helm Repo
Install
CRDs
```

---

# Q3. Scale Down o3db

### What is being tested?

Controllers.

Pods are rarely created directly.

Usually:

```text
Deployment
StatefulSet
ReplicaSet
DaemonSet
Job
```

If you delete Pod:

```bash
kubectl delete pod
```

controller recreates it.

So first identify:

```bash
ownerReferences
```

Then scale controller.

### Exam Pattern

Frequently:

```text
There are 5 pods.
Reduce to 2.
```

Answer is NEVER deleting pods manually.

---

# Q4. Which Pods Die First?

### What is being tested?

QoS Classes.

Kubernetes eviction priority:

```text
BestEffort
↓
Burstable
↓
Guaranteed
```

### BestEffort

No requests

No limits

```yaml
resources: {}
```

Dies first.

### Burstable

Requests defined.

```yaml
requests:
```

### Guaranteed

Requests = Limits

```yaml
requests:
 cpu: 100m

limits:
 cpu: 100m
```

Protected most.

### Memory Trick

Think:

```text
BestEffort = Begging
Burstable = Basic
Guaranteed = VIP
```

---

# Q5. HPA

### What is being tested?

Autoscaling.

HPA watches:

```text
CPU
Memory
Custom metrics
```

Example:

```yaml
averageUtilization: 50
```

means:

```text
Target CPU = 50%
```

If actual:

```text
80%
```

scale up.

If actual:

```text
10%
```

scale down.

### Why min/max replicas?

Prevents:

```text
0 pods
1000 pods
```

situations.

---

# Q6. PV PVC Deployment

### What is being tested?

Storage binding.

Think:

```text
PV = Supply
PVC = Demand
```

Example:

```text
PV = 10Gi available
PVC = asks 2Gi
```

Binding occurs.

### Why storageClassName: ""

Without it:

```text
Dynamic provisioning may occur
```

Question wants static binding.

So:

```yaml
storageClassName: ""
```

on both.

### Mental Model

```text
Application
   ↓
PVC
   ↓
PV
   ↓
Disk
```

---

# Q7. Metrics Server

### What is being tested?

Cluster monitoring basics.

Metrics Server provides:

```bash
kubectl top nodes
kubectl top pods
```

without Prometheus.

### Difference

```bash
kubectl get pods
```

shows state.

```bash
kubectl top pods
```

shows resource usage.

Example:

```text
CPU
Memory
```

---

# Q8. Upgrade Worker Node

### What is being tested?

Cluster lifecycle.

Rule:

```text
Worker version
≤
Control Plane version
```

Worker cannot be newer.

### Upgrade Order

```text
Control Plane
↓
Workers
```

Always.

### Why join?

Node object may exist:

```text
NotReady
```

because kubelet isn't registered.

Join command:

```bash
kubeadm join
```

adds node into cluster.

---

# Q9. ServiceAccount API Access

### What is being tested?

Authentication.

Every Pod automatically gets:

```text
Token
CA certificate
Namespace
```

mounted under:

```text
/var/run/secrets/kubernetes.io/serviceaccount/
```

### Flow

Pod

↓

ServiceAccount

↓

JWT Token

↓

API Server

Example:

```bash
curl \
-H "Authorization: Bearer TOKEN"
```

### Real-world use

Controllers like:

* ArgoCD
* ExternalDNS
* Cert Manager

all do this.

---

# Q10. RBAC

### What is being tested?

Authorization.

Authentication asks:

```text
Who are you?
```

Authorization asks:

```text
What can you do?
```

### Flow

```text
ServiceAccount
↓
Role
↓
RoleBinding
```

Role:

```yaml
verbs:
- create
```

Resources:

```yaml
- secrets
- configmaps
```

Means:

```text
Can create only
Cannot delete
Cannot update
Cannot get
```

---

# Q11. DaemonSet

### What is being tested?

One Pod per Node.

Deployment:

```text
Number of replicas
```

DaemonSet:

```text
One Pod per node
```

Examples:

* Fluentd
* Datadog
* Prometheus Node Exporter

### Why toleration?

Control Plane usually has:

```text
NoSchedule
```

taint.

Without toleration:

```text
DaemonSet skips control plane
```

Question specifically wants:

```text
All nodes
```

so toleration needed.

---

# Q12. AntiAffinity

### What is being tested?

Pod placement.

Question:

```text
3 replicas
2 workers
```

Requirement:

```text
Only one pod per node
```

Result:

```text
Worker1 -> Pod1
Worker2 -> Pod2
Pending -> Pod3
```

### Why?

AntiAffinity says:

```text
Don't place same app on same node
```

Scheduler obeys.

No free node:

```text
Pending
```

---

# Q13. Gateway API

### What is being tested?

Modern replacement for Ingress.

Old:

```text
Ingress
```

New:

```text
Gateway
HTTPRoute
```

### Relationship

```text
Gateway
   ↓
HTTPRoute
   ↓
Service
   ↓
Pod
```

### Header Routing

This question adds:

```text
User-Agent=mobile
```

Route to:

```text
mobile service
```

Otherwise:

```text
desktop service
```

This is Layer-7 routing.

---

# Q14. Certificates

### What is being tested?

Cluster security.

Kubernetes certificates usually expire after:

```text
1 year
```

If expired:

```text
kubectl stops working
```

API Server may fail.

### Check

OpenSSL:

```bash
openssl x509
```

Kubeadm:

```bash
kubeadm certs check-expiration
```

### Renew

```bash
kubeadm certs renew apiserver
```

Very common CKA topic.

---

# Q15. NetworkPolicy

### What is being tested?

Micro-segmentation.

Without policy:

```text
All Pods
↔
All Pods
```

can communicate.

Question wants:

```text
backend
↓
db1:1111

backend
↓
db2:2222
```

ONLY.

Everything else blocked.

### Mental Picture

```text
backend
   ├── db1:1111 ✔
   ├── db2:2222 ✔
   └── vault:3333 ✘
```

---

# Q16. CoreDNS

### What is being tested?

Cluster DNS.

Normal:

```text
service.namespace.svc.cluster.local
```

Question wants:

```text
service.namespace.svc.custom-domain
```

also working.

### Why rewrite?

CoreDNS receives:

```text
custom-domain
```

and rewrites it internally:

```text
cluster.local
```

Application doesn't notice.

### CoreDNS Exam Rule

Always:

```bash
kubectl get cm coredns -n kube-system -o yaml > backup.yaml
```

before editing.

---

# Q17. crictl Investigation

### What is being tested?

Container Runtime Interface (CRI).

Kubernetes:

```text
kubectl
```

talks to:

```text
API Server
```

But node actually runs containers through:

```text
containerd
```

### crictl

Equivalent of Docker commands:

| Docker         | crictl         |
| -------------- | -------------- |
| docker ps      | crictl ps      |
| docker logs    | crictl logs    |
| docker inspect | crictl inspect |

### What is runtimeType?

Shows runtime used:

```text
io.containerd.runc.v2
```

or similar.

### Investigation Flow

```text
Pod
↓
Node
↓
SSH
↓
crictl ps
↓
crictl inspect
↓
crictl logs
```

---

# Most Common CKA Mindset

Before touching the keyboard, classify the question:

| If Question Mentions | Immediately Think |
| -------------------- | ----------------- |
| Pod count            | scale             |
| Storage              | PV/PVC            |
| Permissions          | RBAC              |
| ServiceAccount       | Token/API         |
| Network              | NetworkPolicy     |
| One Pod per Node     | DaemonSet         |
| Separate Pods        | AntiAffinity      |
| Autoscaling          | HPA               |
| Certificates         | kubeadm certs     |
| DNS                  | CoreDNS           |
| Worker upgrade       | kubeadm join      |
| Container runtime    | crictl            |

This classification alone can save several minutes per question during the exam.
