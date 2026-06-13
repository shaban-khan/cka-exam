# CKA Services & Networking (20%) – Complete Theory + Examples + Exam Tips

This is one of the most tested domains because Kubernetes is fundamentally a distributed system.

Think of Networking in layers:

```text
Container
    ↓
Pod
    ↓
Service
    ↓
Ingress/Gateway
    ↓
User
```

Most networking questions can be solved by following that path.

---

# 1. Understand Connectivity Between Pods

# Theory

In Kubernetes, every Pod gets its own IP address.

Example:

```text
nginx-pod   → 10.244.1.10
api-pod     → 10.244.2.15
```

Unlike Docker:

```text
Containers share host networking rules
```

Kubernetes guarantees:

```text
Every Pod can communicate with every other Pod
```

without NAT.

This is called:

```text
Flat Network Model
```

---

## Why?

Suppose:

```text
Frontend Pod
Backend Pod
Database Pod
```

Need communication:

```text
Frontend → Backend
Backend → Database
```

Kubernetes networking enables this automatically.

---

## How?

Through the CNI plugin:

Examples:

* Calico
* Cilium
* Flannel

---

## Example

Get Pod IPs:

```bash
kubectl get pod -o wide
```

Output:

```text
nginx    10.244.1.10
api      10.244.2.15
```

Test:

```bash
kubectl exec nginx -- curl 10.244.2.15
```

---

## Common Troubleshooting

Check:

```bash
kubectl get pods -o wide
```

Verify:

```bash
kubectl exec pod1 -- ping <pod-ip>
```

or

```bash
kubectl exec pod1 -- curl <pod-ip>
```

---

## CKA Tip

When Service isn't working:

```text
First verify Pod-to-Pod communication.
```

---

# 2. Define and Enforce Network Policies

# Theory

By default:

```text
All Pods can talk to all Pods
```

Example:

```text
Frontend → Backend
Frontend → Database
Backend → Database
```

Everything allowed.

---

## Why Network Policies?

Security.

Example:

```text
Frontend should access Backend

Frontend should NOT access Database
```

Need firewall rules for Pods.

---

## NetworkPolicy

Controls:

```text
Ingress
Egress
```

---

## Ingress

Controls:

```text
Who can reach me?
```

Example:

```yaml
policyTypes:
- Ingress
```

---

## Egress

Controls:

```text
Where can I go?
```

Example:

```yaml
policyTypes:
- Egress
```

---

## Deny All Ingress

```yaml
podSelector:
  matchLabels:
    app: nginx
policyTypes:
- Ingress
ingress: []
```

Meaning:

```text
Nobody can reach nginx Pods.
```

---

## Allow Only Frontend

```yaml
from:
- podSelector:
    matchLabels:
      app: frontend
```

Meaning:

```text
Only frontend Pods may connect.
```

---

## CKA Tip

Always identify:

```text
1. Which Pods are selected?
2. Is it Ingress or Egress?
3. Who is allowed?
```

---

## Troubleshooting

```bash
kubectl get netpol
kubectl describe netpol
```

---

# 3. Use ClusterIP, NodePort, LoadBalancer and Endpoints

# Theory

Pods are temporary.

Their IPs change.

Need stable access.

Solution:

```text
Service
```

---

# ClusterIP

Default service type.

```yaml
type: ClusterIP
```

---

## Purpose

Internal communication.

Example:

```text
Frontend → Backend
```

Service gets:

```text
10.96.0.10
```

stable forever.

---

## Flow

```text
Frontend
    ↓
Service
    ↓
Pods
```

---

## NodePort

Exposes application on node.

Example:

```yaml
type: NodePort
```

Node:

```text
192.168.1.10
```

Port:

```text
30080
```

Access:

```text
http://192.168.1.10:30080
```

---

## LoadBalancer

Cloud environments:

* AKS
* EKS
* GKE

Creates:

```text
External Load Balancer
```

Example:

```yaml
type: LoadBalancer
```

Access:

```text
Internet
   ↓
Load Balancer
   ↓
Service
   ↓
Pods
```

---

# Endpoints

Very important for CKA.

Question:

```text
Which Pods does Service send traffic to?
```

Answer:

```text
Endpoints
```

---

## Example

Service selector:

```yaml
selector:
  app: nginx
```

Pod:

```yaml
labels:
  app: nginx
```

Endpoint created:

```bash
kubectl get endpoints
```

Output:

```text
nginx-service
10.244.1.10:80
```

---

## Common Failure

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
No Endpoints
```

---

## Troubleshooting Flow

```bash
kubectl get svc

kubectl describe svc

kubectl get endpoints
```

---

## CKA Tip

Most Service issues are:

```text
Selector mismatch
```

Check endpoints immediately.

---

# 4. Use Gateway API to Manage Ingress Traffic

# Theory

Gateway API is the modern replacement for traditional Ingress.

Old:

```text
Ingress
```

New:

```text
Gateway API
```

---

## Why Gateway API?

Ingress limitations:

```text
Limited extensibility
Limited routing flexibility
```

Gateway API provides:

```text
Gateway
GatewayClass
HTTPRoute
TCPRoute
```

---

## Flow

```text
User
  ↓
Gateway
  ↓
HTTPRoute
  ↓
Service
  ↓
Pods
```

---

## Components

### GatewayClass

Defines implementation.

Example:

```text
Envoy Gateway
```

---

### Gateway

Defines listener.

Example:

```text
Port 80
Port 443
```

---

### HTTPRoute

Defines routing.

Example:

```text
/app → app-service
/api → api-service
```

---

## CKA Tip

Know relationship:

```text
GatewayClass
     ↓
Gateway
     ↓
HTTPRoute
     ↓
Service
```

---

# 5. Ingress Controllers and Ingress Resources

# Theory

Ingress provides:

```text
HTTP Routing
SSL Termination
Host Routing
Path Routing
```

---

## Why Not Use NodePort?

Without Ingress:

```text
Frontend → Port 30080
API      → Port 30081
Admin    → Port 30082
```

Messy.

---

## With Ingress

```text
app.company.com
api.company.com
```

Single entry point.

---

# Ingress Controller

Ingress resource alone does nothing.

Need controller.

Examples:

* NGINX Ingress Controller
* Envoy Gateway
* Traefik

---

## Flow

```text
Internet
    ↓
Ingress Controller
    ↓
Ingress Resource
    ↓
Service
    ↓
Pod
```

---

## Example

```yaml
rules:
- host: app.example.com
```

Meaning:

```text
Requests for app.example.com
go to service
```

---

## Verify

```bash
kubectl get ingress
```

---

## Troubleshooting

```bash
kubectl describe ingress
```

Check:

```text
Backend
Host
Address
```

---

# 6. Understand and Use CoreDNS

# Theory

Pods communicate using names.

Example:

```text
mysql
redis
backend
```

Not IPs.

Need DNS.

---

## CoreDNS

Provides cluster DNS.

Runs as Pods:

```bash
kubectl get pods -n kube-system
```

Example:

```text
coredns-xxxx
```

---

## How DNS Works

Service:

```text
mysql
```

CoreDNS resolves:

```text
mysql.default.svc.cluster.local
```

to

```text
10.96.0.15
```

---

## Flow

```text
Application
     ↓
CoreDNS
     ↓
Service IP
```

---

## DNS Names

Short:

```text
mysql
```

Namespace:

```text
mysql.default
```

Full:

```text
mysql.default.svc.cluster.local
```

---

## Test DNS

Create Pod:

```bash
kubectl run test --image=busybox -it --rm
```

Inside:

```bash
nslookup kubernetes.default
```

Expected:

```text
DNS resolution successful
```

---

## Troubleshooting

Check CoreDNS:

```bash
kubectl get pods -n kube-system
```

Check logs:

```bash
kubectl logs -n kube-system <coredns-pod>
```

---

## Common Failure

Service exists:

```text
mysql
```

Application:

```text
Cannot resolve mysql
```

Check:

```bash
nslookup mysql
```

---

# CKA Networking Troubleshooting Workflow

When an application is unreachable:

### Step 1

Check Pod:

```bash
kubectl get pod -o wide
```

---

### Step 2

Check Service:

```bash
kubectl get svc
```

---

### Step 3

Check Endpoints

```bash
kubectl get endpoints
```

---

### Step 4

Check DNS

```bash
nslookup service-name
```

---

### Step 5

Check Network Policies

```bash
kubectl get netpol
```

---

### Step 6

Check Ingress/Gateway

```bash
kubectl get ingress
kubectl get gateway
kubectl get httproute
```

---

# Most Important CKA Commands

### Pod Connectivity

```bash
kubectl get pod -o wide
kubectl exec
```

### Services

```bash
kubectl get svc
kubectl describe svc
kubectl get endpoints
```

### DNS

```bash
nslookup
```

### Network Policies

```bash
kubectl get netpol
kubectl describe netpol
```

### Ingress

```bash
kubectl get ingress
kubectl describe ingress
```

### Gateway API

```bash
kubectl get gatewayclass
kubectl get gateway
kubectl get httproute
```

---

# Final Memory Sheet

```text
Pod Connectivity
  Every Pod gets an IP

NetworkPolicy
  Kubernetes Firewall

ClusterIP
  Internal Access

NodePort
  Node Access

LoadBalancer
  External Access

Endpoints
  Actual Pods behind Service

Ingress
  HTTP Routing

Gateway API
  Modern Ingress

CoreDNS
  Service Discovery
```

If you master **Service → Endpoints → DNS → NetworkPolicy → Ingress/Gateway troubleshooting**, you'll be able to solve the majority of networking questions that appear on the CKA exam.
