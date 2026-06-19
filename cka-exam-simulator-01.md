This looks very much like a KodeKloud/CKA simulator set. For CKA, speed matters more than perfect YAML writing. Below are the **fastest exam approaches**, the **kubernetes.io docs section to search**, and **time-saving commands**.

---

# 1. Kubeconfig Extraction

### Fast Docs Search

Search:

> kubectl config view

Official doc:
[kubectl config view](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_config/kubectl_config_view/?utm_source=chatgpt.com)

### Solution

List contexts:

```bash
kubectl config get-contexts -o name \
--kubeconfig /opt/course/1/kubeconfig \
> /opt/course/1/contexts
```

Current context:

```bash
kubectl config current-context \
--kubeconfig /opt/course/1/kubeconfig \
> /opt/course/1/current-context
```

Certificate:

```bash
kubectl config view \
--kubeconfig /opt/course/1/kubeconfig \
-o jsonpath='{.users[?(@.name=="account-0027")].user.client-certificate-data}' \
| base64 -d \
> /opt/course/1/cert
```

### Exam Tip

Whenever kubeconfig question appears:

```bash
kubectl config view
kubectl config get-contexts
kubectl config current-context
```

---

# 2. Cert Manager Installation

### Fast Docs Search

Search:

```text
cert-manager helm install crds
```

Official:

[cert-manager Helm Installation](https://cert-manager.io/docs/installation/helm/?utm_source=chatgpt.com)

### Solution

```bash
kubectl create ns cert-manager
```

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

```bash
helm install cert-manager \
jetstack/cert-manager \
-n cert-manager \
--set crds.enabled=true
```

Edit:

```yaml
spec:
  selfSigned:
    crlDistributionPoints:
    - http://example.com/crl
```

Apply:

```bash
kubectl apply -f /opt/course/2/cluster-issuer.yaml
```

### Exam Tip

Remember:

```bash
helm install NAME CHART
```

---

# 3. Scale Down o3db

### Fast Search

```text
kubectl scale deployment
```

### Find owner

```bash
kubectl -n project-h800 get pod
```

Check owner:

```bash
kubectl -n project-h800 get pod o3db-xxx -o yaml
```

Scale:

```bash
kubectl -n project-h800 scale deployment o3db --replicas=1
```

or

```bash
kubectl -n project-h800 scale sts o3db --replicas=1
```

### Exam Tip

Always check owner:

```bash
kubectl get pod POD -o yaml | grep ownerReferences -A5
```

---

# 4. Pods Terminated First

### Fast Search

```text
pod qos class
```

Official:

[Pod QoS Classes](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/?utm_source=chatgpt.com)

### Solution

Find BestEffort Pods:

```bash
kubectl get pod -n project-c13 \
-o custom-columns=NAME:.metadata.name,QOS:.status.qosClass
```

Write BestEffort pod names.

```bash
kubectl get pod -n project-c13 \
-o custom-columns=NAME:.metadata.name,QOS:.status.qosClass
```

### Exam Tip

Eviction order:

```text
BestEffort
Burstable
Guaranteed
```

---

# 5. Kustomize HPA

### Fast Search

```text
kustomize patches
```

Official:

[Kustomize Task](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/?utm_source=chatgpt.com)

### Solution

Delete ConfigMap from kustomization.

Add HPA yaml:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  minReplicas: 2
  maxReplicas: 4
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Prod overlay:

```yaml
maxReplicas: 6
```

Apply:

```bash
kubectl kustomize staging | kubectl apply -f -
kubectl kustomize prod | kubectl apply -f -
```

### Exam Tip

Always:

```bash
grep -R apiVersion .
```

to quickly find kustomize files.

---

# 6. PV PVC Deployment

### Fast Search

```text
persistent volume claim example
```

Official:

[Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/?utm_source=chatgpt.com)

### Solution

PV:

```yaml
capacity:
  storage: 2Gi
accessModes:
- ReadWriteOnce
hostPath:
  path: /Volumes/Data
storageClassName: ""
```

PVC:

```yaml
resources:
  requests:
    storage: 2Gi
storageClassName: ""
```

Deployment:

```yaml
image: httpd:2-alpine
mountPath: /tmp/safari-data
```

### Exam Tip

For static binding:

```yaml
storageClassName: ""
```

on BOTH PV and PVC.

---

# 7. Metrics Scripts

### Solution

node.sh

```bash
#!/bin/bash
kubectl top nodes
```

pod.sh

```bash
#!/bin/bash
kubectl top pods --containers -A
```

### Fast Search

```text
kubectl top pods containers
```

---

# 8. Upgrade Worker Node

### Fast Search

```text
kubeadm upgrade node worker
```

Official:

[Upgrade Kubernetes Cluster](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/?utm_source=chatgpt.com)

### Workflow

Controlplane version:

```bash
kubectl version
```

SSH:

```bash
ssh cka3962-node1
```

Install exact version:

```bash
apt-cache madison kubeadm
```

```bash
apt install kubeadm=<VERSION>
```

Join:

```bash
kubeadm token create --print-join-command
```

Run on worker.

### Exam Tip

Memorize:

```bash
kubeadm token create --print-join-command
```

---

# 9. Read Secrets via API

### Fast Search

```text
serviceaccount token kubernetes api curl
```

### Solution

Create Pod:

```yaml
serviceAccountName: secret-reader
```

Exec:

```bash
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
```

```bash
curl -k \
-H "Authorization: Bearer $TOKEN" \
https://kubernetes.default/api/v1/namespaces/project-swan/secrets
```

Save result:

```bash
> /opt/course/9/result.json
```

### Exam Tip

Useful variables:

```bash
TOKEN=
CACERT=
```

---

# 10. SA Role RoleBinding

### Fast Search

```text
rbac role create secrets configmaps
```

Official:

[RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/?utm_source=chatgpt.com)

Role:

```yaml
resources:
- secrets
- configmaps

verbs:
- create
```

Bind to SA processor.

### Exam Tip

Quick generator:

```bash
kubectl create role
```

---

# 11. DaemonSet

### Fast Search

```text
daemonset toleration control plane
```

Official:

[DaemonSet Concept](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/?utm_source=chatgpt.com)

Need toleration:

```yaml
- key: node-role.kubernetes.io/control-plane
  operator: Exists
  effect: NoSchedule
```

Resources:

```yaml
requests:
  cpu: 10m
  memory: 10Mi
```

### Exam Tip

Fastest:

```bash
kubectl create deployment ds-important \
--image=httpd:2-alpine \
--dry-run=client -o yaml
```

Convert Deployment → DaemonSet.

---

# 12. Deployment With Pod AntiAffinity

### Fast Search

```text
requiredDuringSchedulingIgnoredDuringExecution hostname
```

Official:

[Assign Pods to Nodes using Affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/?utm_source=chatgpt.com)

Key:

```yaml
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
  - labelSelector:
      matchExpressions:
      - key: id
        operator: In
        values:
        - very-important
    topologyKey: kubernetes.io/hostname
```

### Exam Tip

This is the classic:

```text
3 replicas
2 workers
1 Pending
```

Question.

---

# 13. Gateway API

### Fast Search

```text
httproute backendrefs matches headers
```

Official:

[Gateway API HTTPRoute](https://gateway-api.sigs.k8s.io/api-types/httproute/?utm_source=chatgpt.com)

Key idea:

```yaml
matches:
- path:
    type: PathPrefix
    value: /auto
  headers:
  - type: Exact
    name: User-Agent
    value: mobile
```

Backend:

```yaml
backendRefs:
- name: mobile
```

Second rule:

```yaml
/auto
```

without header → desktop.

### Exam Tip

Always inspect existing Gateway:

```bash
kubectl get gateway -A
```

---

# 14. Certificate Expiry

### Fast Search

```text
kubeadm certs check-expiration
```

Official:

[kubeadm certs check-expiration](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-certs/?utm_source=chatgpt.com)

Check:

```bash
openssl x509 -in \
/etc/kubernetes/pki/apiserver.crt \
-noout -enddate
```

Compare:

```bash
kubeadm certs check-expiration
```

Renew command file:

```bash
echo "kubeadm certs renew apiserver" \
> /opt/course/14/kubeadm-renew-certs.sh
```

### Exam Tip

Memorize:

```bash
kubeadm certs check-expiration
kubeadm certs renew apiserver
```

---

# 15. NetworkPolicy

### Fast Search

```text
network policy egress ports podselector
```

Official:

[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/?utm_source=chatgpt.com)

Use:

```yaml
policyTypes:
- Egress
```

Allow:

```yaml
to:
- podSelector:
    matchLabels:
      app: db1
  ports:
  - port: 1111
```

and

```yaml
app: db2
port: 2222
```

### Exam Tip

Start from:

```bash
kubectl create networkpolicy --help
```

---

# 16. CoreDNS

### Fast Search

```text
coredns rewrite plugin
```

Official:

[CoreDNS Plugins Overview](https://coredns.io/plugins/?utm_source=chatgpt.com)

Backup:

```bash
kubectl get cm coredns \
-n kube-system \
-o yaml \
> /opt/course/16/coredns_backup.yaml
```

Add rewrite:

```text
rewrite name suffix custom-domain cluster.local
```

Restart:

```bash
kubectl rollout restart deployment coredns -n kube-system
```

### Exam Tip

Always backup first.

---

# 17. crictl Investigation

### Fast Search

```text
crictl ps inspect logs
```

Official:

[crictl User Guide](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md?utm_source=chatgpt.com)

Create Pod.

Find node:

```bash
kubectl get pod -o wide
```

SSH node.

Container:

```bash
sudo crictl ps
```

Inspect:

```bash
sudo crictl inspect CONTAINER_ID
```

Logs:

```bash
sudo crictl logs CONTAINER_ID
```

runtimeType:

```bash
sudo crictl inspect CONTAINER_ID | grep runtimeType
```

### Exam Tip

Most common commands:

```bash
crictl ps
crictl inspect
crictl logs
```

---

# CKA Last-Minute Search Strategy

During the exam, use kubernetes.io search with only these keywords:

| Task           | Search                    |
| -------------- | ------------------------- |
| PV/PVC         | persistent volume claim   |
| HPA            | horizontal pod autoscaler |
| RBAC           | role rolebinding          |
| NetworkPolicy  | network policy egress     |
| DaemonSet      | daemonset                 |
| Affinity       | pod anti affinity         |
| Gateway        | httproute                 |
| Upgrade        | kubeadm upgrade           |
| Certs          | kubeadm certs             |
| CoreDNS        | coredns rewrite           |
| Storage        | hostpath pv               |
| ServiceAccount | service account token     |

These searches usually get you to the correct page in under 10 seconds during the CKA exam.
