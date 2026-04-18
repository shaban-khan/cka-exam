---

# 🚀 Certified Kubernetes Administrator (CKA) — COMPLETE CHEAT SHEET

---

# ⚡ 0. Time-Saving Setup (DO THIS FIRST)

```bash
alias k=kubectl
export do="--dry-run=client -o yaml"
```

👉 Use:

```bash
k run nginx --image=nginx $do > pod.yaml
```

---

# 🧱 1. Pods (Core Building Block)

## Create Pod (fast way)

```bash
k run nginx --image=nginx
```

## YAML Template

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    env: prod
spec:
  containers:
  - name: c1
    image: nginx
```

## Debug

```bash
k describe pod <pod>
k logs <pod>
k exec -it <pod> -- sh
```

---

# 📦 2. Deployments

```bash
k create deployment nginx --image=nginx
k scale deployment nginx --replicas=3
k set image deployment/nginx nginx=nginx:1.25
k rollout undo deployment nginx
k rollout status deployment nginx
```

👉 **Exam tip:** Rollback saves marks when update fails

---

# 🌐 3. Services

```bash
k expose deployment nginx --port=80 --type=ClusterIP
k expose deployment nginx --port=80 --type=NodePort
```

## Test from pod

```bash
k run test --image=busybox -it --rm -- sh
wget -qO- http://<service-name>
```

👉 Always use **service name, not IP**

---

# ⚙️ 4. ConfigMaps

```bash
k create configmap app --from-literal=mode=dev
```

## YAML (env)

```yaml
env:
- name: MODE
  valueFrom:
    configMapKeyRef:
      name: app
      key: mode
```

---

# 🔐 5. Secrets

```bash
k create secret generic db --from-literal=pwd=1234
```

👉 View:

```bash
k get secret db -o yaml
```

⚠️ Base64 ≠ secure encryption

---

# 🔑 6. RBAC (VERY IMPORTANT)

## Role YAML

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list"]
```

## Bind Role

```bash
k create rolebinding rb --role=reader --serviceaccount=default:sa
```

## Test

```bash
k auth can-i get pods --as=system:serviceaccount:default:sa
```

---

# 💾 7. Storage

## emptyDir (easy marks)

```yaml
volumes:
- name: data
  emptyDir: {}
```

## PVC Template

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

👉 Mount:

```yaml
volumeMounts:
- mountPath: /data
  name: data
```

---

# 📍 8. Scheduling

## NodeSelector

```bash
k label nodes node1 env=prod
```

```yaml
nodeSelector:
  env: prod
```

---

## Taints

```bash
k taint nodes node1 key=value:NoSchedule
```

## Tolerations

```yaml
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

---

# 🧠 9. Troubleshooting (MOST IMPORTANT)

## Always run:

```bash
k describe pod <pod>
k logs <pod>
k get events
```

## Common Issues:

* Image typo ❌
* Port mismatch ❌
* Missing config ❌
* CrashLoopBackOff ❌

👉 Fix = read error, don’t guess

---

# 🛠️ 10. Cluster Maintenance

```bash
k cordon node1
k drain node1 --ignore-daemonsets
k uncordon node1
```

---

# 🧠 11. etcd (Basic Commands)

```bash
ETCDCTL_API=3 etcdctl snapshot save backup.db
ETCDCTL_API=3 etcdctl snapshot restore backup.db
```

👉 Just know flow (not deep theory)

---

# ⚡ 12. YAML Shortcuts (LIFE SAVER)

```bash
k run nginx --image=nginx $do > pod.yaml
k create deployment nginx --image=nginx $do > deploy.yaml
```

---

# 📚 13. Documentation Strategy (CRITICAL)

👉 Use official docs smartly:

* Search:
  **"kubernetes pod yaml example"**
* Go directly to:

  * Tasks → Workloads
  * RBAC section
  * Storage section

⏱️ Target: find answer in <30 sec

---

# 🧪 14. Exam Strategy (MOST IMPORTANT)

### 🔥 During Exam:

* Do **easy questions first**
* Skip if stuck >5 mins
* Don’t panic if something breaks

### ⚡ Speed Tricks:

* Use YAML generation (`--dry-run`)
* Use `kubectl explain`
* Copy from docs → modify

---

# ❌ Common Mistakes (Avoid These)

* Writing YAML from memory (use dry-run instead)
* Spending too long on one question
* Forgetting namespace
* Not checking errors properly

---

# 🧠 Final Mindset

* You are NOT expected to know everything
* You ARE expected to:

  * debug fast
  * use docs
  * stay calm

---

# 🎯 Final Suggestion (Most Honest Advice)

If you want to **maximize passing chances**:

👉 Do this once before exam:

* 1 full mock (2 hours)
* 1 troubleshooting session (fix broken pods)
* 1 RBAC practice

That alone can increase your score by **20–30%**

---
