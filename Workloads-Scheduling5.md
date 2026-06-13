This is a common **CKA exam objective**. Here's a concise cheat sheet for **Pod admission and scheduling**.

---

# 1. Resource Requests and Limits

### Pod with CPU and Memory Requests/Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "256Mi"
```

Verify:

```bash
kubectl describe pod nginx
```

---

# 2. LimitRange

Sets default requests/limits in a namespace.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    type: Container
```

Apply:

```bash
kubectl apply -f limitrange.yaml
```

---

# 3. ResourceQuota

Restricts total resource usage in a namespace.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    limits.cpu: "4"
    requests.memory: 2Gi
    limits.memory: 4Gi
```

Check:

```bash
kubectl describe quota
```

---

# 4. Node Selector

Schedule on nodes with specific labels.

Label node:

```bash
kubectl label node worker1 disktype=ssd
```

Pod:

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

---

# 5. Node Affinity

More flexible than nodeSelector.

### Required

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```

### Preferred

```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1
      preference:
        matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```

---

# 6. Taints

Prevent scheduling.

Add taint:

```bash
kubectl taint nodes worker1 env=prod:NoSchedule
```

View:

```bash
kubectl describe node worker1 | grep Taints
```

---

# 7. Tolerations

Allow pod onto tainted node.

```yaml
tolerations:
- key: "env"
  operator: "Equal"
  value: "prod"
  effect: "NoSchedule"
```

---

# 8. Taints and Tolerations Example

Node:

```bash
kubectl taint node worker1 app=db:NoSchedule
```

Pod:

```yaml
tolerations:
- key: app
  operator: Equal
  value: db
  effect: NoSchedule
```

Result:

```text
Pod can run on worker1
```

---

# 9. Pod Affinity

Schedule near another pod.

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - frontend
      topologyKey: kubernetes.io/hostname
```

---

# 10. Pod Anti-Affinity

Avoid same node.

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - frontend
      topologyKey: kubernetes.io/hostname
```

Useful for HA.

---

# 11. Check Why Pod Isn't Scheduled

Most useful CKA command:

```bash
kubectl describe pod <pod-name>
```

Look at:

```text
Events:
  FailedScheduling
```

Example:

```text
0/3 nodes available:
1 node(s) had taint,
2 node(s) didn't match node affinity
```

---

# 12. Fast CKA Commands

```bash
kubectl get nodes --show-labels

kubectl label node worker1 disktype=ssd

kubectl taint node worker1 env=prod:NoSchedule

kubectl describe node worker1

kubectl describe pod nginx

kubectl get quota

kubectl get limitrange
```

---

### CKA Exam Topics Most Frequently Tested

1. Resource Requests & Limits
2. ResourceQuota
3. LimitRange
4. NodeSelector
5. Node Affinity
6. Taints & Tolerations
7. Troubleshooting FailedScheduling

These cover the majority of "Configure Pod admission and scheduling" tasks you'll encounter.
