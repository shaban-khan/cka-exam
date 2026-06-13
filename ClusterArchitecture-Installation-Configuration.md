This domain is worth **25% of the CKA exam** and is one of the hardest because it combines:

```text
Cluster Administration
+
Architecture
+
Security
+
Installation
+
Troubleshooting
```

For CKA, don't just memorize commands. Understand **why Kubernetes is designed this way**.

---

# 1. Manage Role Based Access Control (RBAC)

# Theory

By default, Kubernetes is a shared system.

Imagine:

```text
Admin
Developer
Tester
Auditor
```

All using the same cluster.

Question:

```text
Who can do what?
```

RBAC answers this.

---

## Why RBAC?

Without RBAC:

```text
Developer deletes production namespace
```

Disaster.

Need:

```text
Developer:
  Read Pods

Admin:
  Full Access
```

---

## RBAC Components

```text
Role
   ↓
RoleBinding
   ↓
User / ServiceAccount
```

---

## Role

Defines permissions.

Example:

```yaml
kind: Role
```

Meaning:

```text
What actions are allowed?
```

---

## RoleBinding

Connects:

```text
Role
  ↓
User
```

---

## Example

Allow reading Pods:

```text
get
list
watch
```

But not:

```text
delete
```

---

## Common Commands

```bash
kubectl get roles
kubectl get rolebindings
kubectl auth can-i get pods
kubectl auth can-i delete pods
```

---

## CKA Tip

Very common task:

```text
Create ServiceAccount
Create Role
Create RoleBinding
Verify access
```

---

## Sample Exam Question

```text
Create a ServiceAccount named app-sa.

Allow it to get/list pods in namespace app1.
```

---

# 2. Prepare Infrastructure for Installing Kubernetes

# Theory

Before Kubernetes starts, infrastructure must exist.

Requirements:

```text
Machines
Network
CPU
Memory
Container Runtime
```

---

## Components Needed

Control Plane:

```text
kube-apiserver
etcd
scheduler
controller-manager
```

Workers:

```text
kubelet
kube-proxy
container runtime
```

---

## Typical kubeadm Requirements

Disable swap:

```bash
swapoff -a
```

Why?

Kubernetes requires predictable memory behavior.

---

## Verify Runtime

```bash
crictl info
```

or

```bash
systemctl status containerd
```

---

## CKA Tip

Memorize:

```text
Swap Off
Container Runtime
Network Connectivity
```

---

# 3. Create and Manage Clusters Using kubeadm

# Theory

kubeadm is the official Kubernetes bootstrap tool.

Without kubeadm:

```text
Install everything manually
```

Very difficult.

---

## What kubeadm Does

Creates:

```text
Certificates
Control Plane Components
Kubeconfig Files
```

Automatically.

---

## Cluster Creation Flow

### Initialize

```bash
kubeadm init
```

Creates:

```text
API Server
Scheduler
Controller Manager
etcd
```

---

### Configure kubectl

```bash
mkdir -p $HOME/.kube

cp /etc/kubernetes/admin.conf ~/.kube/config
```

---

### Join Worker

```bash
kubeadm token create --print-join-command
```

Output:

```bash
kubeadm join ...
```

Run on worker node.

---

## Verify

```bash
kubectl get nodes
```

---

## CKA Tip

Memorize:

```bash
kubeadm init

kubeadm token create --print-join-command

kubectl get nodes
```

---

# 4. Manage Cluster Lifecycle

# Theory

Clusters evolve.

Need:

```text
Upgrade
Patch
Backup
Restore
```

---

## Upgrade Flow

Check version:

```bash
kubeadm version
```

Plan:

```bash
kubeadm upgrade plan
```

Apply:

```bash
kubeadm upgrade apply
```

---

## Worker Upgrade

Drain:

```bash
kubectl drain worker1
```

Upgrade.

Uncordon:

```bash
kubectl uncordon worker1
```

---

## Drain Theory

Before maintenance:

```text
Move workloads away
```

Node becomes:

```text
Unschedulable
```

---

## Common Commands

```bash
kubectl drain node1

kubectl cordon node1

kubectl uncordon node1
```

---

## Sample Question

```text
Upgrade control plane from v1.32.x to v1.33.x.
```

---

# 5. Implement Highly Available Control Plane

# Theory

Single control plane:

```text
Control Plane Down
      ↓
Cluster Management Lost
```

Need redundancy.

---

## HA Architecture

```text
Load Balancer
      ↓
CP1
CP2
CP3
```

---

## Why?

If:

```text
CP1 fails
```

Then:

```text
CP2 continues
```

---

## etcd HA

Can run:

```text
Stacked
```

or

```text
External
```

Topology.

---

## CKA Tip

Know concepts:

```text
Load Balancer
Multiple Control Planes
Quorum
```

Mostly theory.

---

# 6. Use Helm and Kustomize

---

# Helm Theory

Helm is:

```text
Package Manager for Kubernetes
```

Like:

```text
apt
yum
npm
```

for Kubernetes.

---

## Problem Helm Solves

Without Helm:

```text
100 YAML files
```

Need:

```text
Install
Upgrade
Rollback
```

Easily.

---

## Common Commands

Add repo:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Install:

```bash
helm install nginx bitnami/nginx
```

List:

```bash
helm list
```

Upgrade:

```bash
helm upgrade
```

Rollback:

```bash
helm rollback
```

---

## Kustomize Theory

Customize YAML without editing originals.

Base:

```text
deployment.yaml
service.yaml
```

Overlay:

```text
dev
prod
```

---

## Example

Change:

```text
replicas: 2
```

to

```text
replicas: 5
```

without modifying base file.

---

## Commands

```bash
kubectl kustomize .

kubectl apply -k .
```

---

# 7. Understand Extension Interfaces

Very important theory.

---

# CRI

Container Runtime Interface

Purpose:

```text
Kubernetes
      ↓
Container Runtime
```

Examples:

```text
containerd
CRI-O
```

---

## Why?

Kubernetes doesn't run containers directly.

It talks through CRI.

---

# CNI

Container Network Interface

Purpose:

```text
Pod Networking
```

Examples:

```text
Calico
Cilium
Flannel
Weave
```

Provides:

```text
Pod IP
Routing
Network Policies
```

---

# CSI

Container Storage Interface

Purpose:

```text
Storage Integration
```

Examples:

```text
Azure Disk
EBS
Ceph
NFS
```

Provides:

```text
PV
PVC
StorageClass
```

---

## CKA Tip

Memorize:

```text
CRI → Containers
CNI → Networking
CSI → Storage
```

---

# 8. Understand CRDs and Operators

Very important modern Kubernetes topic.

---

# CRD Theory

Normally Kubernetes understands:

```text
Pod
Deployment
Service
```

Suppose you want:

```text
Database
```

Kubernetes doesn't know that resource.

CRD allows:

```text
Create new API objects
```

---

## Example

Create:

```text
MySQL
```

as a Kubernetes object.

```yaml
kind: MySQL
```

This becomes possible using a CRD.

---

# Operator Theory

Problem:

CRD only defines object type.

Who manages it?

Need controller.

Operator =

```text
CRD
 +
Controller Logic
```

---

## Example

Create:

```yaml
kind: MySQL
```

Operator automatically:

```text
Creates Pods
Backups
Scaling
Recovery
Upgrades
```

---

## Famous Operators

Examples:

* Prometheus Operator
* Strimzi
* Percona Operator

---

## CKA Tip

Understand:

```text
CRD = New API Type

Operator = Automation for that API Type
```

---

# Most Important Commands for This Domain

### RBAC

```bash
kubectl auth can-i
kubectl get role
kubectl get rolebinding
```

### kubeadm

```bash
kubeadm init

kubeadm join

kubeadm token create --print-join-command
```

### Lifecycle

```bash
kubectl cordon
kubectl drain
kubectl uncordon

kubeadm upgrade plan
```

### Helm

```bash
helm install
helm list
helm upgrade
helm rollback
```

### Kustomize

```bash
kubectl apply -k .
```

---

# CKA Exam Questions You Should Practice

### RBAC

```text
Create a ServiceAccount and grant read access to Pods.
```

### kubeadm

```text
Add a worker node to the cluster.
```

### Lifecycle

```text
Upgrade a node and return it to service.
```

### Helm

```text
Install an application using a Helm chart.
```

### Networking

```text
Identify which CNI plugin is installed.
```

### Storage

```text
Identify which CSI driver is used.
```

### CRD

```text
List custom resources in the cluster.
```

Useful commands:

```bash
kubectl get crd

kubectl api-resources
```

---

# Final Memory Sheet

```text
RBAC
  Who can do what?

kubeadm
  Build cluster

Lifecycle
  Upgrade cluster

HA
  Multiple control planes

Helm
  Package manager

Kustomize
  YAML customization

CRI
  Containers

CNI
  Networking

CSI
  Storage

CRD
  New Kubernetes API

Operator
  Automation for CRDs
```

If you master this section together with Troubleshooting and Storage, you'll cover the bulk of the administration-heavy tasks that commonly appear in the CKA exam.
