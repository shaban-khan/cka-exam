# CKA Storage - Complete Theory + Examples + Troubleshooting + Exam Tips

Storage is a very common CKA topic because Kubernetes Pods are **ephemeral**.

When a Pod dies:

```text
Pod Deleted
    ↓
Container Files Lost
```

Storage solves this problem.

---

# Big Picture

Without Storage:

```text
Pod
 └─ Data
```

Pod deleted:

```text
Data Lost
```

With Storage:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage
```

Pod deleted:

```text
Data Survives
```

---

# 1. Implement Storage Classes and Dynamic Volume Provisioning

# Theory

Before StorageClasses, an administrator had to manually create storage.

Example:

```text
Admin creates disk
    ↓
Creates PV
    ↓
Developer creates PVC
```

This is called:

```text
Static Provisioning
```

Problem:

```text
Manual
Slow
Not scalable
```

---

# Dynamic Provisioning

Instead:

```text
PVC Created
     ↓
StorageClass
     ↓
PV Automatically Created
```

No admin intervention.

---

# StorageClass

A StorageClass defines:

```text
Which storage to use
How to create it
```

Think:

```text
Storage Template
```

---

# Example

StorageClass:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/no-provisioner
```

In cloud:

```text
AWS EBS
Azure Disk
GCE PD
```

StorageClass creates disks automatically.

---

# How Dynamic Provisioning Works

```text
PVC Created
     ↓
StorageClass Selected
     ↓
PV Created Automatically
     ↓
PVC Bound
```

---

# Verify

```bash
kubectl get sc
```

Example:

```text
NAME                 PROVISIONER
default              disk.csi.azure.com
fast                 disk.csi.azure.com
```

---

# CKA Tip

Always check:

```bash
kubectl get sc
```

before troubleshooting PVC issues.

---

# Sample Question

```text
Create a PVC that uses the fast StorageClass.
```

---

# 2. Configure Volume Types, Access Modes and Reclaim Policies

# Theory

A Volume defines:

```text
Where data is stored
How data is accessed
```

---

# Common Volume Types

---

## emptyDir

Temporary storage.

```yaml
volumes:
- name: cache
  emptyDir: {}
```

Lifecycle:

```text
Pod Created
 ↓
Storage Exists

Pod Deleted
 ↓
Storage Deleted
```

Use:

```text
Cache
Scratch Data
```

---

## hostPath

Mount node filesystem.

```yaml
hostPath:
  path: /data
```

Example:

```text
Node
 └─ /data
      ↓
Pod
```

Good for labs.

Not recommended for production.

---

## Persistent Volume

Storage exists independently of Pod.

```text
PV
 ↓
PVC
 ↓
Pod
```

Most important for CKA.

---

# Access Modes

Determines:

```text
Who can use storage?
```

---

## ReadWriteOnce (RWO)

```text
One Node
Read + Write
```

Example:

```text
Node1
 └─ Pod1
```

Most common.

Azure Disk:

```text
RWO
```

---

## ReadOnlyMany (ROX)

```text
Many Nodes
Read Only
```

Example:

```text
Pod1
Pod2
Pod3
```

Can read.

Cannot write.

---

## ReadWriteMany (RWX)

```text
Many Nodes
Read + Write
```

Example:

```text
Pod1
Pod2
Pod3
```

All can write.

Typical:

```text
NFS
Azure Files
```

---

# Reclaim Policies

Question:

```text
What happens when PVC is deleted?
```

---

## Delete

```yaml
persistentVolumeReclaimPolicy: Delete
```

Flow:

```text
PVC Deleted
 ↓
PV Deleted
 ↓
Storage Deleted
```

Common in cloud.

---

## Retain

```yaml
persistentVolumeReclaimPolicy: Retain
```

Flow:

```text
PVC Deleted
 ↓
PV Remains
 ↓
Data Remains
```

Used for:

```text
Databases
Critical Data
```

---

## Recycle

Deprecated.

Ignore for CKA.

---

# Verify

```bash
kubectl get pv
```

Output:

```text
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY
pv1      1Gi        RWO            Retain
```

---

# CKA Tip

Memorize:

```text
RWO = One Node RW
ROX = Many Nodes Read
RWX = Many Nodes RW

Delete = Remove Data
Retain = Keep Data
```

---

# Sample Question

```text
Create a PV with:
1Gi
ReadWriteOnce
Retain policy
```

---

# 3. Manage Persistent Volumes and Persistent Volume Claims

# Theory

Most important storage topic in CKA.

---

# Persistent Volume (PV)

PV is actual storage.

Think:

```text
Storage Resource
```

Created by:

```text
Administrator
OR
StorageClass
```

Example:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 1Gi
```

---

# Persistent Volume Claim (PVC)

Developer requests storage.

Think:

```text
Storage Request
```

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

---

# Relationship

```text
PV = Storage Supply

PVC = Storage Demand
```

---

# Binding Process

Step 1

PV exists:

```text
1Gi
RWO
```

Step 2

PVC requests:

```text
500Mi
RWO
```

Step 3

Kubernetes binds:

```text
PVC → PV
```

Status:

```text
Bound
```

---

# Check Binding

PV:

```bash
kubectl get pv
```

PVC:

```bash
kubectl get pvc
```

Example:

```text
NAME   STATUS
pvc1   Bound
```

---

# Use PVC in Pod

```yaml
volumes:
- name: data
  persistentVolumeClaim:
    claimName: pvc1
```

Mount:

```yaml
volumeMounts:
- mountPath: /data
  name: data
```

---

# Complete Flow

```text
StorageClass
      ↓
Persistent Volume
      ↓
Persistent Volume Claim
      ↓
Pod
```

or

```text
PVC
 ↓
StorageClass
 ↓
PV Created Automatically
 ↓
Pod
```

---

# Common Troubleshooting

## PVC Pending

Check:

```bash
kubectl describe pvc pvc1
```

Common causes:

### No StorageClass

```text
No default StorageClass
```

### No Matching PV

PVC:

```text
10Gi
```

PV:

```text
1Gi
```

Cannot bind.

---

### Access Mode Mismatch

PVC:

```text
RWX
```

PV:

```text
RWO
```

Cannot bind.

---

# Most Important Commands

Check StorageClass:

```bash
kubectl get sc
```

Check PV:

```bash
kubectl get pv
```

Check PVC:

```bash
kubectl get pvc
```

Describe PVC:

```bash
kubectl describe pvc
```

Describe PV:

```bash
kubectl describe pv
```

---

# CKA Exam Tips

## Tip 1

Always visualize:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
StorageClass
```

---

## Tip 2

When PVC is Pending:

```bash
kubectl describe pvc
```

FIRST.

---

## Tip 3

Check:

```bash
kubectl get sc
kubectl get pv
kubectl get pvc
```

before editing YAML.

---

## Tip 4

Memorize binding requirements:

```text
Capacity
Access Mode
StorageClass
```

must match.

---

# CKA Quick Revision Sheet

### StorageClass

```text
Storage Template
Dynamic Provisioning
```

---

### PV

```text
Actual Storage
```

---

### PVC

```text
Storage Request
```

---

### Dynamic Provisioning

```text
PVC
 ↓
StorageClass
 ↓
PV Created Automatically
```

---

### Access Modes

```text
RWO = One Node RW
ROX = Many Nodes Read
RWX = Many Nodes RW
```

---

### Reclaim Policies

```text
Delete = Remove Storage
Retain = Keep Storage
```

---

### Most Important Commands

```bash
kubectl get sc

kubectl get pv

kubectl get pvc

kubectl describe pvc

kubectl describe pv
```

---

# Typical CKA Storage Questions

### Question 1

```text
Create a PV:
1Gi
RWO
Retain
```

### Question 2

```text
Create a PVC:
500Mi
Use existing PV
```

### Question 3

```text
Mount PVC into Pod at /data
```

### Question 4

```text
PVC remains Pending.
Find and fix the issue.
```

### Question 5

```text
Create a StorageClass and dynamically provision storage.
```

If you master **PV → PVC → StorageClass → Pod**, plus **RWO/RWX** and **Delete/Retain**, you'll be well prepared for nearly all storage-related tasks in the CKA exam.
