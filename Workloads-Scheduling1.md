# For the **CKA exam**, the Deployment topic is extremely important because many tasks involve:

* Create a Deployment
* Scale it
* Update container images
* Perform rolling updates
* Check rollout status
* Roll back to previous versions
* Troubleshoot failed deployments

---

# 1. What is a Deployment?

A Deployment manages ReplicaSets and Pods.

```text
Deployment
     ↓
 ReplicaSet
     ↓
    Pods
```

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
```

Create:

```bash
kubectl apply -f deploy.yaml
```

---

# 2. Fast Deployment Creation (CKA Favorite)

Create deployment:

```bash
kubectl create deployment nginx --image=nginx
```

Generate YAML:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```

Create with replicas:

```bash
kubectl create deployment nginx --image=nginx --replicas=3
```

---

# 3. Check Deployment

```bash
kubectl get deploy
kubectl get rs
kubectl get po
```

Detailed:

```bash
kubectl describe deploy nginx
```

---

# 4. Scale Deployment

Current:

```text
3 replicas
```

Scale:

```bash
kubectl scale deployment nginx --replicas=5
```

Verify:

```bash
kubectl get deploy
```

Output:

```text
READY   UP-TO-DATE   AVAILABLE
5/5
```

---

# 5. Rolling Update

Current image:

```text
nginx:1.25
```

Update:

```bash
kubectl set image deployment/nginx \
nginx=nginx:1.26
```

Format:

```bash
kubectl set image deployment/<deployment-name> \
<container-name>=<image>
```

Example:

```bash
kubectl set image deployment/web \
web=nginx:1.26
```

---

# 6. Monitor Rollout

Very common in CKA.

```bash
kubectl rollout status deployment nginx
```

Output:

```text
deployment "nginx" successfully rolled out
```

Watch:

```bash
kubectl get pods -w
```

You will see:

```text
Old Pod Terminating
New Pod Creating
```

---

# 7. What Happens During Rolling Update?

Suppose:

```text
Replicas = 3
```

Current:

```text
Pod1 nginx:1.25
Pod2 nginx:1.25
Pod3 nginx:1.25
```

Update:

```text
nginx:1.26
```

Kubernetes:

```text
Create New Pod
Wait Ready
Delete Old Pod

Create New Pod
Wait Ready
Delete Old Pod

Create New Pod
Wait Ready
Delete Old Pod
```

Result:

```text
No downtime
```

---

# 8. Rollout History

View revisions:

```bash
kubectl rollout history deployment nginx
```

Output:

```text
REVISION
1
2
3
```

Detailed:

```bash
kubectl rollout history deployment nginx --revision=2
```

---

# 9. Rollback

Rollback to previous version:

```bash
kubectl rollout undo deployment nginx
```

Verify:

```bash
kubectl rollout status deployment nginx
```

---

# 10. Rollback to Specific Revision

See history:

```bash
kubectl rollout history deployment nginx
```

Rollback:

```bash
kubectl rollout undo deployment nginx --to-revision=2
```

---

# 11. Deployment Strategy

Default:

```yaml
strategy:
  type: RollingUpdate
```

Explicit:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

---

## maxUnavailable

```yaml
maxUnavailable: 1
```

Means:

```text
At most 1 pod can be unavailable
```

Example:

```text
3 replicas

2 must remain available
```

---

## maxSurge

```yaml
maxSurge: 1
```

Means:

```text
Create 1 extra pod temporarily
```

Example:

```text
Desired replicas = 3

Can temporarily run 4 pods
```

---

# 12. Recreate Strategy

```yaml
strategy:
  type: Recreate
```

Behavior:

```text
Delete all old pods
Then create new pods
```

Result:

```text
Possible downtime
```

---

# 13. Pause Rollout

Pause:

```bash
kubectl rollout pause deployment nginx
```

Check:

```bash
kubectl describe deployment nginx
```

Resume:

```bash
kubectl rollout resume deployment nginx
```

---

# 14. Restart Deployment

CKA favorite.

Restart all pods:

```bash
kubectl rollout restart deployment nginx
```

Useful after:

* ConfigMap changes
* Secret changes

---

# 15. Troubleshooting Failed Deployment

Check deployment:

```bash
kubectl describe deployment nginx
```

Check pods:

```bash
kubectl get pods
```

Check logs:

```bash
kubectl logs <pod>
```

Events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# 16. Important Exam Commands

### Create Deployment

```bash
kubectl create deployment nginx --image=nginx
```

### Scale

```bash
kubectl scale deployment nginx --replicas=5
```

### Update Image

```bash
kubectl set image deployment/nginx nginx=nginx:1.26
```

### Check Rollout

```bash
kubectl rollout status deployment nginx
```

### History

```bash
kubectl rollout history deployment nginx
```

### Rollback

```bash
kubectl rollout undo deployment nginx
```

### Rollback Specific Revision

```bash
kubectl rollout undo deployment nginx --to-revision=2
```

### Restart

```bash
kubectl rollout restart deployment nginx
```

---

# CKA Exam Scenario Practice

### Task 1

Create deployment:

```bash
kubectl create deployment web --image=nginx:1.25 --replicas=3
```

### Task 2

Upgrade:

```bash
kubectl set image deployment/web nginx=nginx:1.26
```

### Task 3

Verify:

```bash
kubectl rollout status deployment web
```

### Task 4

Check history:

```bash
kubectl rollout history deployment web
```

### Task 5

Rollback:

```bash
kubectl rollout undo deployment web
```

### Task 6

Verify image:

```bash
kubectl get pods -o wide
kubectl describe deployment web
```

---

## Must Memorize for CKA

```bash
kubectl create deployment nginx --image=nginx

kubectl scale deployment nginx --replicas=5

kubectl set image deployment/nginx nginx=nginx:1.26

kubectl rollout status deployment nginx

kubectl rollout history deployment nginx

kubectl rollout undo deployment nginx

kubectl rollout restart deployment nginx
```

If you can perform those commands without looking them up, you're well prepared for almost every Deployment-related task in the CKA exam.
