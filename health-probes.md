In Kubernetes, there are **3 probe types** (health checks).

| Probe Type      | Purpose                                | Action                                           |
| --------------- | -------------------------------------- | ------------------------------------------------ |
| Readiness Probe | Can the Pod receive traffic?           | Add/Remove from Service endpoints                |
| Liveness Probe  | Is the container still alive?          | Restart container if failed                      |
| Startup Probe   | Has the application finished starting? | Prevent liveness/readiness checks during startup |

---

# 1. Readiness Probe

Question:

```text id="d9t1bx"
Can I send traffic to this Pod?
```

If failed:

```text id="ah6m8j"
Ready=False
```

Service stops sending traffic.

Example:

```yaml id="ayvtt5"
readinessProbe:
  httpGet:
    path: /health
    port: 8080
```

---

# 2. Liveness Probe

Question:

```text id="jgwl2f"
Is the application still alive?
```

If failed:

```text id="h8m4pc"
Container Restarted
```

Example:

```yaml id="2bz9bk"
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

Flow:

```text id="pqfd1l"
Probe Failed
      |
      v
Kubelet
      |
      v
Restart Container
```

---

# 3. Startup Probe

Question:

```text id="1d0g2n"
Has the application finished starting?
```

Useful for:

```text id="fpp9c8"
Java Apps
Spring Boot
Large Applications
```

which take a long time to start.

Example:

```yaml id="p7gh0f"
startupProbe:
  httpGet:
    path: /health
    port: 8080
```

Without startup probe:

```text id="1h98oq"
Application takes 2 minutes
Liveness runs after 30 seconds
Liveness fails
Container restarted
```

Infinite loop.

Startup probe prevents this.

---

# Another Classification (How Probe Checks Health)

Each probe can use **3 methods**:

| Method     | Example                      |
| ---------- | ---------------------------- |
| HTTP GET   | GET /health                  |
| TCP Socket | Check port open              |
| Exec       | Run command inside container |

Example:

### HTTP

```yaml id="sknrw2"
httpGet:
  path: /health
  port: 8080
```

---

### TCP

```yaml id="y4st8r"
tcpSocket:
  port: 8080
```

---

### Exec

```yaml id="4shtj3"
exec:
  command:
  - cat
  - /tmp/healthy
```

---

# Easy CKA Memory

## Probe Types

```text id="6lhm4d"
Readiness
Liveness
Startup
```

Remember:

```text id="tqh5ns"
Readiness = Traffic
Liveness  = Restart
Startup   = Boot-up
```

---

## Probe Methods

```text id="n8ry3v"
HTTP
TCP
Exec
```
