# All 3 probes that is useful for CKA.

---

# 1. Readiness Probe

### Question Kubernetes Asks

```text id="d8u6z4"
Can I send traffic to this Pod?
```

### Example Output

```text id="w4j8q2"
Readiness:
http-get http://:80/
delay=5s
timeout=1s
period=10s
#success=1
#failure=3
```

### Meaning

| Field      | Meaning                            |
| ---------- | ---------------------------------- |
| delay=5s   | Wait 5 seconds after start         |
| timeout=1s | Probe must respond within 1 second |
| period=10s | Check every 10 seconds             |
| success=1  | 1 success = Ready                  |
| failure=3  | 3 failures = Not Ready             |

### Flow

```text id="p6m3k1"
Pod Starts
    |
    v
Wait 5s
    |
    v
GET PodIP:80/
    |
    +--> Success
    |      |
    |      v
    |   READY=1/1
    |   Service sends traffic
    |
    +--> Fail 3 times
           |
           v
        READY=0/1
        No Service traffic
```

### Effect

```text id="r7y4n2"
Service adds/removes Pod from Endpoints
```

---

# 2. Liveness Probe

### Question Kubernetes Asks

```text id="m2t8c5"
Is the application still alive?
```

### Example Output

```text id="x5n7b4"
Liveness:
http-get http://:80/
delay=10s
timeout=1s
period=10s
#success=1
#failure=3
```

### Meaning

| Field      | Meaning                            |
| ---------- | ---------------------------------- |
| delay=10s  | Wait 10 seconds before first check |
| period=10s | Check every 10 seconds             |
| failure=3  | Restart after 3 failures           |

### Flow

```text id="k8z3f1"
Application Running
        |
        v
Probe every 10s
        |
        +--> Success
        |
        +--> Fail
               |
               v
        Fail 3 times
               |
               v
        Kubelet Restarts Container
```

### Effect

```text id="c9p4w6"
Container Restart
```

### Verify

```bash id="h2v7x9"
kubectl get pods
```

You may see:

```text id="s4q1m8"
RESTARTS=5
```

---

# 3. Startup Probe

### Question Kubernetes Asks

```text id="j6k3r7"
Has the application finished starting?
```

### Example Output

```text id="n8y2d4"
Startup:
http-get http://:80/
delay=0s
timeout=1s
period=10s
#success=1
#failure=30
```

### Meaning

| Field      | Meaning                          |
| ---------- | -------------------------------- |
| period=10s | Check every 10 seconds           |
| failure=30 | Allow 30 failures before killing |

Maximum startup time:

```text id="t1w6g9"
30 × 10 seconds
=
300 seconds
=
5 minutes
```

### Flow

```text id="u3e8h2"
Container Starts
       |
       v
Startup Probe Runs
       |
       +--> Fail
       |      |
       |      v
       |   Keep Waiting
       |
       +--> Success
              |
              v
      Enable Readiness Probe
      Enable Liveness Probe
```

### Effect

```text id="b7r5k3"
Protects slow-starting applications
```

---

# Real World Example

Suppose:

```text id="q4m9v1"
Spring Boot App
```

Needs:

```text id="y2n8f6"
90 seconds
```

to start.

Without Startup Probe:

```text id="e5c3p7"
Container Starts
     |
     v
Liveness runs after 30s
     |
     v
Fails
     |
     v
Restart
```

Infinite restart loop.

---

With Startup Probe:

```text id="d7k1r8"
Container Starts
      |
      v
Startup Probe waits
      |
      v
Application Ready
      |
      v
Startup Success
      |
      v
Readiness + Liveness begin
```

---

# YAML Examples

## Readiness

```yaml id="g8m4t2"
readinessProbe:
  httpGet:
    path: /
    port: 80
```

---

## Liveness

```yaml id="w9n6c3"
livenessProbe:
  httpGet:
    path: /
    port: 80
```

---

## Startup

```yaml id="p2k7x4"
startupProbe:
  httpGet:
    path: /
    port: 80
```

---

# CKA Exam Memory Table

| Probe     | Question             | Action                   |
| --------- | -------------------- | ------------------------ |
| Readiness | Can receive traffic? | Remove/Add from Service  |
| Liveness  | Is process healthy?  | Restart Container        |
| Startup   | Finished booting?    | Delay readiness/liveness |

### One-Line Memory Trick

```text id="z6r3n5"
Readiness = Traffic
Liveness  = Restart
Startup   = Boot-up
```

If you remember only that line, you'll answer most probe-related CKA questions correctly.
