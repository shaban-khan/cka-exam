

---

# 🚀 SUPER HARD CKA SET 1 (Tasks 1–10)

---

## 🔥 Task 1 — Broken Pod + ConfigMap Fix

### 🎯 Problem:

A pod `web` is crashing because it expects env `APP_MODE` from ConfigMap `app-config` key `mode`.

### ✅ Solution:

```bash
kubectl create configmap app-config --from-literal=mode=prod
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
  - name: web
    image: nginx
    env:
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: mode
```

---

## 🔥 Task 2 — Deployment Rollback Under Failure

### 🎯 Problem:

Deployment updated to bad image → fix quickly

### ✅ Solution:

```bash
kubectl rollout undo deployment nginx
```

👉 Or:

```bash
kubectl set image deployment/nginx nginx=nginx:latest
```

---

## 🔥 Task 3 — RBAC Read-Only User

### 🎯 Problem:

User can only list/get pods in namespace `dev`

### ✅ Solution:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: readonly
  namespace: dev
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: bind
  namespace: dev
subjects:
- kind: ServiceAccount
  name: readonly
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 🔥 Task 4 — PVC Binding Issue

### 🎯 Problem:

PVC not binding → fix storage

### ✅ Solution:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

## 🔥 Task 5 — Node Scheduling Restriction

### 🎯 Problem:

Pod must run only on node labeled `env=prod`

### ✅ Solution:

```bash
kubectl label nodes node1 env=prod
```

```yaml
nodeSelector:
  env: prod
```

---

## 🔥 Task 6 — Taint + Toleration Control

### 🎯 Problem:

Only specific pod should run on tainted node

### ✅ Solution:

```bash
kubectl taint nodes node1 key=value:NoSchedule
```

```yaml
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

---

## 🔥 Task 7 — Service Not Reachable (Debug)

### 🎯 Problem:

Service exists but not reachable

### ✅ Fix Steps:

```bash
kubectl get endpoints
kubectl describe svc <svc>
```

👉 Ensure:

* Labels match
* Port matches container port

---

## 🔥 Task 8 — CrashLoopBackOff Debug

### 🎯 Problem:

Pod crashing continuously

### ✅ Solution:

```bash
kubectl logs <pod>
kubectl describe pod <pod>
```

👉 Fix:

* Wrong command
* Missing env
* Bad image

---

## 🔥 Task 9 — Multi-Container Communication

### 🎯 Problem:

2 containers share data

### ✅ Solution:

```yaml
volumes:
- name: shared
  emptyDir: {}

containers:
- name: c1
  image: nginx
  volumeMounts:
  - mountPath: /data
    name: shared

- name: c2
  image: busybox
  volumeMounts:
  - mountPath: /data
    name: shared
```

---

## 🔥 Task 10 — Node Drain + Recovery

### 🎯 Problem:

Safely move workloads off node

### ✅ Solution:

```bash
kubectl drain node1 --ignore-daemonsets
kubectl uncordon node1
```

---

# ⚡ How to Use These (IMPORTANT)

Don’t just read.

👉 Do this:

1. Set timer (60–90 mins)
2. Solve all 10
3. No notes
4. Only docs

---

# 🧠 Reality Check

If you can:

* Solve 8/10 without help
  👉 You’re exam ready

If not:
👉 Focus on weak area (RBAC / troubleshooting)

---

## 🔥 Task 11 — Fix Wrong Selector (Service Bug)

### 🎯 Problem:

Service `web-svc` exists but has **no endpoints**

### 🧠 Hint:

Selector mismatch

### ✅ Solution:

```bash
kubectl get svc web-svc -o yaml
kubectl get pods --show-labels
```

👉 Fix:

```bash
kubectl edit svc web-svc
```

✔️ Ensure:

```yaml
selector:
  app: nginx
```

---

## 🔥 Task 12 — Deployment Not Scaling (HPA Trap)

### 🎯 Problem:

Deployment doesn’t scale even after setting replicas

### 🧠 Hint:

Check autoscaler conflict

### ✅ Solution:

```bash
kubectl get hpa
kubectl delete hpa <name>
```

👉 Then:

```bash
kubectl scale deployment nginx --replicas=5
```

---

## 🔥 Task 13 — Secret Not Working (Mount Issue)

### 🎯 Problem:

Secret mounted but file missing

### ✅ Solution:

```bash
kubectl describe pod <pod>
```

👉 Fix YAML:

```yaml
volumes:
- name: secret-vol
  secret:
    secretName: db-secret

volumeMounts:
- mountPath: /etc/secret
  name: secret-vol
```

---

## 🔥 Task 14 — Pod Stuck Pending (Scheduling Failure)

### 🎯 Problem:

Pod stuck in `Pending`

### 🧠 Causes:

* NodeSelector mismatch
* Taint issue

### ✅ Solution:

```bash
kubectl describe pod <pod>
kubectl get nodes --show-labels
```

👉 Fix label or remove selector

---

## 🔥 Task 15 — Create Job (One-Time Task)

### 🎯 Problem:

Run a pod that completes and exits

### ✅ Solution:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job1
spec:
  template:
    spec:
      containers:
      - name: c1
        image: busybox
        command: ["sh","-c","echo hello"]
      restartPolicy: Never
```

---

## 🔥 Task 16 — CronJob Creation

### 🎯 Problem:

Run job every minute

### ✅ Solution:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron1
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: c1
            image: busybox
            command: ["sh","-c","date"]
          restartPolicy: Never
```

---

## 🔥 Task 17 — Limit Resource Usage

### 🎯 Problem:

Restrict CPU & memory

### ✅ Solution:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "200m"
    memory: "256Mi"
```

---

## 🔥 Task 18 — Fix ImagePullBackOff

### 🎯 Problem:

Pod cannot pull image

### 🧠 Causes:

* Wrong image name
* Private registry

### ✅ Solution:

```bash
kubectl describe pod <pod>
```

👉 Fix image:

```bash
kubectl edit pod <pod>
```

---

## 🔥 Task 19 — NetworkPolicy (Basic Restriction)

### 🎯 Problem:

Allow traffic only from specific pods

### ✅ Solution:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: client
```

---

## 🔥 Task 20 — Debug Init Container Failure

### 🎯 Problem:

Pod stuck because init container fails

### ✅ Solution:

```bash
kubectl logs <pod> -c <init-container>
```

👉 Fix command or config

---

# ⚡ Key Learning from Set 2

👉 These are **real traps**:

* Selector mismatch
* HPA conflict
* Secret mount errors
* Pending pods
* Init container failures

---

## 🔥 Task 21 — Multi-Issue Pod (Image + Port + Env)

### 🎯 Problem:

Pod `api` is not reachable:

* Wrong image tag
* Container port mismatch
* Missing env from ConfigMap

### ✅ Fix Steps:

```bash
kubectl describe pod api
kubectl logs api
```

👉 Fix all three:

* Image → valid tag (e.g., `nginx:latest`)
* Container port → match service (e.g., 80)
* Add env:

```yaml
env:
- name: MODE
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: mode
```

---

## 🔥 Task 22 — Service + Endpoint Debug Combo

### 🎯 Problem:

Service exists, pods running, still no traffic

### 🧠 Trap:

* Label mismatch OR wrong targetPort

### ✅ Fix:

```bash
kubectl get endpoints svc-name
kubectl describe svc svc-name
```

✔️ Ensure:

* selector matches pod labels
* `targetPort` = containerPort

---

## 🔥 Task 23 — PVC Not Bound (StorageClass Missing)

### 🎯 Problem:

PVC stuck in `Pending`

### 🧠 Cause:

No StorageClass or mismatch

### ✅ Fix:

```bash
kubectl get storageclass
```

👉 Use existing:

```yaml
storageClassName: standard
```

OR remove field if static PV exists

---

## 🔥 Task 24 — etcd Backup (REAL COMMAND)

### 🎯 Problem:

Take etcd snapshot

### ✅ Solution:

```bash
ETCDCTL_API=3 etcdctl snapshot save /tmp/backup.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=<ca.crt> \
--cert=<server.crt> \
--key=<server.key>
```

👉 Usually certs at:

```
/etc/kubernetes/pki/etcd/
```

---

## 🔥 Task 25 — etcd Restore

### 🎯 Problem:

Restore cluster from snapshot

### ✅ Solution:

```bash
ETCDCTL_API=3 etcdctl snapshot restore /tmp/backup.db \
--data-dir=/var/lib/etcd-new
```

👉 Then update:

* kube-apiserver etcd path

---

## 🔥 Task 26 — Broken Deployment (Readiness Probe)

### 🎯 Problem:

Pods running but not ready

### 🧠 Cause:

Readiness probe failing

### ✅ Fix:

```bash
kubectl describe pod <pod>
```

👉 Fix YAML:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
```

---

## 🔥 Task 27 — Node Not Ready

### 🎯 Problem:

Node shows `NotReady`

### ✅ Fix:

```bash
kubectl get nodes
kubectl describe node <node>
```

👉 Check:

* kubelet
* disk pressure
* network

---

## 🔥 Task 28 — DaemonSet Creation

### 🎯 Problem:

Run pod on ALL nodes

### ✅ Solution:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds
spec:
  selector:
    matchLabels:
      app: ds
  template:
    metadata:
      labels:
        app: ds
    spec:
      containers:
      - name: c1
        image: nginx
```

---

## 🔥 Task 29 — Fix Resource Starvation

### 🎯 Problem:

Pod not scheduled due to insufficient resources

### ✅ Fix:

```bash
kubectl describe pod <pod>
```

👉 Reduce requests:

```yaml
resources:
  requests:
    cpu: "50m"
```

---

## 🔥 Task 30 — Multi-Step Debug (REAL EXAM STYLE)

### 🎯 Problem:

App not working:

* Deployment exists
* Service exists
* Pod crashing

### ✅ Debug Flow:

```bash
kubectl get all
kubectl describe pod
kubectl logs
kubectl get svc
kubectl get endpoints
```

👉 Fix:

* Image
* Config
* Ports
* Labels

---

# ⚡ What This Set Teaches

👉 This is where people fail:

* Multi-layer debugging
* etcd commands
* Storage edge cases
* Probes

---

# 🧠 Reality Check (After Set 3)

If you can:

* Solve ≥7/10
  👉 You are **exam ready**

If not:
👉 Focus on:

* Debugging flow
* Storage
* etcd basics

---

## 🔥 Task 31 — Namespace + RBAC + Deployment (3-in-1)

### 🎯 Problem:

* Create namespace `team-a`
* Deploy nginx inside it
* Create ServiceAccount `sa-a`
* Allow SA to **only list pods in that namespace**

### ✅ Solution:

```bash id="6kg2x2"
kubectl create ns team-a
kubectl create deployment nginx --image=nginx -n team-a
kubectl create sa sa-a -n team-a
```

```yaml id="rxm2qe"
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: team-a
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
```

```bash id="l1l4sh"
kubectl create rolebinding rb \
--role=pod-reader \
--serviceaccount=team-a:sa-a \
-n team-a
```

---

## 🔥 Task 32 — Multi-Container + Shared Config + Volume

### 🎯 Problem:

* Two containers share:

  * ConfigMap
  * Volume

### ✅ Solution:

```yaml id="yq9w8k"
volumes:
- name: shared
  emptyDir: {}
- name: config
  configMap:
    name: app-config

containers:
- name: app
  image: nginx
  volumeMounts:
  - mountPath: /data
    name: shared
  - mountPath: /config
    name: config

- name: sidecar
  image: busybox
  volumeMounts:
  - mountPath: /data
    name: shared
```

---

## 🔥 Task 33 — Fix Broken Service Chain

### 🎯 Problem:

Traffic flow broken:
Pod → Service → Client

### 🧠 Likely Issues:

* Labels mismatch
* Wrong port

### ✅ Debug:

```bash id="l0x4j0"
kubectl get pods --show-labels
kubectl get svc -o yaml
kubectl get endpoints
```

👉 Fix selector + ports

---

## 🔥 Task 34 — Job + Retry Policy

### 🎯 Problem:

Create Job that retries 3 times if fails

### ✅ Solution:

```yaml id="6q9i70"
apiVersion: batch/v1
kind: Job
metadata:
  name: retry-job
spec:
  backoffLimit: 3
  template:
    spec:
      containers:
      - name: c1
        image: busybox
        command: ["false"]
      restartPolicy: Never
```

---

## 🔥 Task 35 — CronJob + Cleanup

### 🎯 Problem:

* Run every minute
* Keep only last 2 jobs

### ✅ Solution:

```yaml id="9pk7zl"
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron
spec:
  schedule: "* * * * *"
  successfulJobsHistoryLimit: 2
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: c1
            image: busybox
            command: ["date"]
          restartPolicy: Never
```

---

## 🔥 Task 36 — Advanced Scheduling Combo

### 🎯 Problem:

* Pod runs ONLY on node with label
* But avoid nodes with another label

### ✅ Solution:

```yaml id="y22v6t"
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: env
          operator: In
          values:
          - prod
        - key: type
          operator: NotIn
          values:
          - gpu
```

---

## 🔥 Task 37 — LimitRange + Resource Control

### 🎯 Problem:

Set default CPU/memory in namespace

### ✅ Solution:

```yaml id="d0x4w4"
apiVersion: v1
kind: LimitRange
metadata:
  name: limits
spec:
  limits:
  - default:
      cpu: "200m"
      memory: "256Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    type: Container
```

---

## 🔥 Task 38 — NetworkPolicy Restriction (2 Layers)

### 🎯 Problem:

* Only allow traffic:

  * from specific namespace
  * to specific pod

### ✅ Solution:

```yaml id="y6p0q1"
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: client
```

---

## 🔥 Task 39 — Fix ImagePullBackOff (Private Registry)

### 🎯 Problem:

Image pull fails

### ✅ Solution:

```bash id="wq7scn"
kubectl create secret docker-registry regcred \
--docker-username=user \
--docker-password=pass \
--docker-server=<registry>
```

```yaml id="8u8c0n"
imagePullSecrets:
- name: regcred
```

---

## 🔥 Task 40 — Full System Debug (REAL EXAM STYLE)

### 🎯 Problem:

Everything exists but app not working:

* Deployment
* Service
* ConfigMap
* Pod

### ✅ Debug Flow (memorize this):

```bash id="n0kz4l"
kubectl get all
kubectl describe pod
kubectl logs
kubectl get svc
kubectl get endpoints
kubectl get configmap
```

👉 Fix:

* Labels
* Ports
* Config
* Image

---


## 🔥 Task 41 — Fix Namespace Resource Quota Block

### 🎯 Problem:

Pods fail to create due to quota limits

### 🧠 Clue:

`Exceeded quota`

### ✅ Solution:

```bash
kubectl describe quota -n <ns>
```

👉 Fix:

* Reduce pod resources OR
* Update quota:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
```

---

## 🔥 Task 42 — Broken Liveness Probe

### 🎯 Problem:

Pod keeps restarting

### 🧠 Cause:

Liveness probe failing

### ✅ Fix:

```bash
kubectl describe pod <pod>
```

👉 Correct:

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 10
```

---

## 🔥 Task 43 — Multi-Container Dependency (Init Container)

### 🎯 Problem:

Main container must wait for condition

### ✅ Solution:

```yaml
initContainers:
- name: init
  image: busybox
  command: ["sh","-c","sleep 10"]
```

---

## 🔥 Task 44 — Fix Wrong Command (CrashLoop)

### 🎯 Problem:

Container exits immediately

### 🧠 Cause:

Bad command

### ✅ Fix:

```yaml
command: ["sh","-c","sleep 3600"]
```

---

## 🔥 Task 45 — Advanced RBAC (Cluster Scope)

### 🎯 Problem:

User can read nodes cluster-wide

### ✅ Solution:

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get","list"]
```

```bash
kubectl create clusterrolebinding crb \
--clusterrole=node-reader \
--user=user1
```

---

## 🔥 Task 46 — Debug DNS Failure

### 🎯 Problem:

Pod cannot resolve service name

### ✅ Debug:

```bash
kubectl exec -it <pod> -- nslookup <service>
```

👉 Check:

* CoreDNS running
* Service exists

---

## 🔥 Task 47 — Fix NodeSelector + Taint Conflict

### 🎯 Problem:

Pod not scheduled even with correct label

### 🧠 Cause:

Node is tainted

### ✅ Fix:

Add toleration:

```yaml
tolerations:
- key: "key"
  operator: "Exists"
  effect: "NoSchedule"
```

---

## 🔥 Task 48 — Rolling Update Failure Recovery

### 🎯 Problem:

Deployment update stuck

### ✅ Fix:

```bash
kubectl rollout undo deployment <name>
kubectl rollout status deployment <name>
```

---

## 🔥 Task 49 — Combine Config + Secret + Volume

### 🎯 Problem:

App needs:

* ConfigMap (env)
* Secret (volume)

### ✅ Solution:

```yaml
env:
- name: MODE
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: mode

volumes:
- name: secret-vol
  secret:
    secretName: db-secret
```

---

## 🔥 Task 50 — Ultimate Debug (Everything Looks Fine)

### 🎯 Problem:

System deployed but not working

### ✅ MASTER DEBUG FLOW (memorize this):

```bash
kubectl get all
kubectl describe pod
kubectl logs
kubectl get svc
kubectl get endpoints
kubectl get configmap
kubectl get secret
kubectl get events
```

👉 Check:

* Labels
* Ports
* Config
* DNS
* Permissions

---

# 🧠 FINAL TRUTH (READ THIS CAREFULLY)

If you can:

* Debug fast
* Use docs quickly
* Fix broken setups

👉 You WILL pass CKA.

---

# ⚡ FINAL 1-HOUR REVISION PLAN (DO BEFORE EXAM)

### ⏱️ Last 60 mins:

**0–15 min**

* Aliases + YAML templates

**15–30 min**

* RBAC + PVC

**30–45 min**

* Troubleshooting commands

**45–60 min**

* 2 mini mock tasks

---

# 🚀 Final Advice (Most Honest)

Don’t aim for 100%.
Aim for:
👉 speed
👉 calmness
👉 consistency

---
