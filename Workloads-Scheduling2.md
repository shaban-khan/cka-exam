# This is a **high-frequency CKA topic**. You should know how to:

1. Create ConfigMaps
2. Create Secrets
3. Consume them as Environment Variables
4. Mount them as Volumes
5. Troubleshoot them

---

# 1. ConfigMap vs Secret

| ConfigMap          | Secret                      |
| ------------------ | --------------------------- |
| Non-sensitive data | Sensitive data              |
| Plain text         | Base64 encoded              |
| App configuration  | Passwords, tokens, API keys |

Examples:

### ConfigMap

```text
APP_COLOR=blue
APP_MODE=prod
```

### Secret

```text
DB_PASSWORD=admin123
API_KEY=xxxxx
```

---

# 2. Create ConfigMap

### Imperative

```bash
kubectl create configmap app-config \
--from-literal=color=blue \
--from-literal=mode=prod
```

View:

```bash
kubectl get configmap
kubectl describe configmap app-config
```

---

### YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  color: blue
  mode: prod
```

Apply:

```bash
kubectl apply -f configmap.yaml
```

---

# 3. Use ConfigMap as Environment Variables

ConfigMap:

```yaml
data:
  color: blue
  mode: prod
```

Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: color
```

Inside pod:

```bash
echo $APP_COLOR
```

Output:

```text
blue
```

---

# 4. Import Entire ConfigMap

Instead of one key:

```yaml
envFrom:
- configMapRef:
    name: app-config
```

Creates:

```text
color=blue
mode=prod
```

as environment variables.

---

# 5. Mount ConfigMap as Volume

ConfigMap:

```yaml
data:
  app.conf: |
    color=blue
```

Pod:

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config

containers:
- name: nginx
  image: nginx
  volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```

Inside pod:

```bash
cat /etc/config/app.conf
```

Output:

```text
color=blue
```

---

# 6. Create Secret

### Imperative

```bash
kubectl create secret generic db-secret \
--from-literal=username=admin \
--from-literal=password=Pass123
```

View:

```bash
kubectl get secret
kubectl describe secret db-secret
```

---

# 7. Secret YAML

Generate:

```bash
kubectl create secret generic db-secret \
--from-literal=username=admin \
--from-literal=password=Pass123 \
--dry-run=client -o yaml
```

Output:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  username: YWRtaW4=
  password: UGFzczEyMw==
```

Notice:

```text
admin → YWRtaW4=
Pass123 → UGFzczEyMw==
```

Base64 encoded.

---

# 8. Decode Secret

Example:

```bash
echo YWRtaW4= | base64 -d
```

Output:

```text
admin
```

Useful during exams.

---

# 9. Use Secret as Environment Variable

```yaml
env:
- name: DB_USER
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: username

- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

Verify:

```bash
kubectl exec -it pod-name -- env | grep DB
```

---

# 10. Import Entire Secret

```yaml
envFrom:
- secretRef:
    name: db-secret
```

Creates:

```text
username=admin
password=Pass123
```

inside container.

---

# 11. Mount Secret as Volume

```yaml
volumes:
- name: secret-volume
  secret:
    secretName: db-secret
```

Mount:

```yaml
volumeMounts:
- name: secret-volume
  mountPath: /etc/secret
```

Inside pod:

```bash
ls /etc/secret
```

Output:

```text
username
password
```

Check:

```bash
cat /etc/secret/username
```

Output:

```text
admin
```

---

# 12. Most Common CKA Commands

Create ConfigMap:

```bash
kubectl create configmap app-config \
--from-literal=color=blue
```

Create Secret:

```bash
kubectl create secret generic db-secret \
--from-literal=password=Pass123
```

View:

```bash
kubectl get cm
kubectl get secret
```

Describe:

```bash
kubectl describe cm app-config
kubectl describe secret db-secret
```

---

# 13. Exam Troubleshooting

### ConfigMap not found

```text
configmap "app-config" not found
```

Check:

```bash
kubectl get cm
```

---

### Secret not found

```text
secret "db-secret" not found
```

Check:

```bash
kubectl get secret
```

---

### Wrong key

ConfigMap:

```yaml
data:
  color: blue
```

Pod:

```yaml
key: colour
```

Result:

```text
CreateContainerConfigError
```

Check:

```bash
kubectl describe pod
```

---

# CKA Memory Sheet

### Create ConfigMap

```bash
kubectl create configmap app-config \
--from-literal=color=blue
```

### Create Secret

```bash
kubectl create secret generic db-secret \
--from-literal=password=Pass123
```

### Use ConfigMap

```yaml
configMapKeyRef
```

### Use Secret

```yaml
secretKeyRef
```

### Mount ConfigMap

```yaml
configMap:
  name: app-config
```

### Mount Secret

```yaml
secret:
  secretName: db-secret
```

### Decode Secret

```bash
echo <base64-value> | base64 -d
```

For the CKA exam, focus on these 4 practical tasks:

1. Create a ConfigMap.
2. Create a Secret.
3. Inject them as environment variables.
4. Mount them as files in a Pod.

Those cover the vast majority of ConfigMap/Secret questions you'll encounter.
