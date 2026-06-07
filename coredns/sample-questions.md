For the **CKA exam**, CoreDNS questions are usually **troubleshooting-based**, not theory-based.

Here are the **Top 5 most likely CoreDNS scenarios** that can appear in the exam.

---

# Question 1: Service Name Resolution Not Working

### Scenario

Application cannot reach service by name.

```bash
curl http://nginx-svc
```

Error:

```text
Could not resolve host: nginx-svc
```

---

### What examiner expects

Verify DNS.

### Commands

```bash
kubectl get pods -n kube-system
```

Check CoreDNS:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

Create debug pod:

```bash
kubectl run test --image=busybox:1.36 -it --rm -- sh
```

Inside:

```bash
nslookup kubernetes.default
```

---

### Root Cause

Usually:

```text
CoreDNS pod crashed
```

or

```text
CoreDNS service missing
```

---

### Fix

Restart CoreDNS:

```bash
kubectl rollout restart deployment coredns -n kube-system
```

---

# Question 2: CoreDNS Pod Not Running

### Scenario

You are told:

```text
Cluster DNS is unavailable.
```

---

### Check

```bash
kubectl get pods -n kube-system
```

Output:

```text
coredns-xxxxx   CrashLoopBackOff
```

---

### Next Step

Check logs:

```bash
kubectl logs -n kube-system deployment/coredns
```

or

```bash
kubectl logs <coredns-pod> -n kube-system
```

---

### Expected Skills

```bash
kubectl describe pod
kubectl logs
```

Find:

```text
Bad Corefile
Image issue
Configuration issue
```

---

# Question 3: Service Exists But DNS Fails

### Scenario

```bash
kubectl get svc
```

shows:

```text
nginx-svc
```

But:

```bash
nslookup nginx-svc
```

fails.

---

### Investigation

Check CoreDNS Service:

```bash
kubectl get svc -n kube-system
```

Expected:

```text
kube-dns
10.96.0.10
```

---

Check Pod DNS:

```bash
kubectl exec -it test -- cat /etc/resolv.conf
```

Expected:

```text
nameserver 10.96.0.10
```

---

### Common Root Cause

Wrong DNS policy.

Example:

```yaml
dnsPolicy: None
```

instead of:

```yaml
dnsPolicy: ClusterFirst
```

---

### Fix

```yaml
dnsPolicy: ClusterFirst
```

---

# Question 4: CoreDNS ConfigMap Broken

### Scenario

Someone modified CoreDNS configuration.

DNS stopped working.

---

### Check

```bash
kubectl get cm coredns -n kube-system -o yaml
```

Look for:

```text
Corefile
```

---

### Example Problem

```text
cluster.local
```

removed accidentally.

or

```text
kubernetes cluster.local
```

missing.

---

### Fix

Restore Corefile.

Restart:

```bash
kubectl rollout restart deployment coredns -n kube-system
```

---

### Very Common Exam Skill

Edit ConfigMap:

```bash
kubectl edit cm coredns -n kube-system
```

---

# Question 5: Service Name Resolves But Application Still Fails

This is a favorite trap.

---

### DNS Works

```bash
nslookup app2-svc
```

Output:

```text
10.96.100.50
```

Looks good.

---

### But Application Fails

```bash
curl app2-svc
```

fails.

---

### Check Endpoints

```bash
kubectl get endpoints
```

Output:

```text
app2-svc   <none>
```

---

### Meaning

DNS is working.

CoreDNS is healthy.

Problem is elsewhere.

---

### Root Cause

Wrong selector.

Example:

Service:

```yaml
selector:
  app: app2
```

Pod:

```yaml
labels:
  app: web
```

---

### Fix

Correct labels/selector.

---

# Commands You Must Memorize

```bash
# CoreDNS Pods
kubectl get pods -n kube-system

# CoreDNS Logs
kubectl logs -n kube-system deployment/coredns

# CoreDNS ConfigMap
kubectl get cm coredns -n kube-system -o yaml

# Edit ConfigMap
kubectl edit cm coredns -n kube-system

# CoreDNS Service
kubectl get svc -n kube-system

# DNS Test Pod
kubectl run test --image=busybox:1.36 -it --rm -- sh

# DNS Resolution Test
nslookup kubernetes.default

# Pod DNS Config
cat /etc/resolv.conf

# Check Service
kubectl get svc

# Check Endpoints
kubectl get endpoints
```

# If I were writing the CKA exam

My most likely CoreDNS practical task would be:

```text
A pod in namespace demo cannot reach service nginx-svc.

Investigate and restore name resolution.
```

Expected troubleshooting sequence:

```text
1. Check Service
2. Check Endpoints
3. Check CoreDNS Pods
4. Check CoreDNS Logs
5. Check /etc/resolv.conf
6. Test with nslookup
7. Fix CoreDNS or DNS policy
```

If you can complete those steps without looking at notes, you're well prepared for almost any CoreDNS-related CKA task.
