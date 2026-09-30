# Day 57 – Resource Requests, Limits, and Probes

## Overview

Today I worked with Kubernetes resource management and container health probes.

The main objectives were:

* Configure CPU and memory requests and limits.
* Understand Kubernetes QoS classes.
* Observe an `OOMKilled` container.
* Create a Pod that remains `Pending` because of insufficient resources.
* Test liveness, readiness, and startup probes.
* Understand how Kubernetes automatically manages unhealthy containers.

---

# Task 1: Resource Requests and Limits

## Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "250m"
          memory: "256Mi"
```

## Apply

```bash
kubectl apply -f resource-pod.yaml
```

Check the Pod:

```bash
kubectl get pod resource-pod
```

Inspect resources:

```bash
kubectl describe pod resource-pod
```

The output contains:

```text
Requests:
  cpu:     100m
  memory:  128Mi

Limits:
  cpu:     250m
  memory:  256Mi
```

## QoS Class

Check:

```bash
kubectl get pod resource-pod -o jsonpath='{.status.qosClass}'
```

Output:

```text
Burstable
```

### Why?

The Pod has resource requests and limits, but they are different:

```text
CPU:
Request = 100m
Limit   = 250m

Memory:
Request = 128Mi
Limit   = 256Mi
```

Therefore, Kubernetes assigns the Pod the:

**QoS Class: Burstable**

### Kubernetes QoS Classes

| QoS Class  | Condition                                                    |
| ---------- | ------------------------------------------------------------ |
| Guaranteed | Requests and limits are defined and equal for all containers |
| Burstable  | Requests/limits exist but do not meet Guaranteed conditions  |
| BestEffort | No requests or limits are defined                            |

### Key Learning

**Requests** are primarily used by the scheduler when deciding where to place a Pod.

**Limits** define the maximum amount of a resource a container can consume.

---

# Task 2: OOMKilled – Exceeding Memory Limits

## Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: oom-pod
spec:
  containers:
    - name: stress
      image: polinux/stress
      resources:
        limits:
          memory: "100Mi"
      command: ["stress"]
      args:
        - "--vm"
        - "1"
        - "--vm-bytes"
        - "200M"
        - "--vm-hang"
        - "1"
```

Apply:

```bash
kubectl apply -f oom-pod.yaml
```

Watch:

```bash
kubectl get pod oom-pod -w
```

Inspect:

```bash
kubectl describe pod oom-pod
```

Look under the container state/previous state for:

```text
Reason: OOMKilled
Exit Code: 137
```

## Why 137?

Linux signal-based exit codes are represented as:

```text
128 + signal number
```

`SIGKILL` is signal `9`.

Therefore:

```text
128 + 9 = 137
```

The container exceeded its memory limit and was terminated with `SIGKILL`.

### Verification

**What exit code does an OOMKilled container have?**

**Answer: `137`**

### CPU vs Memory

CPU is a **compressible resource**.

When a container reaches its CPU limit, CPU usage can be throttled.

Memory is an **incompressible resource**.

When a container attempts to exceed its memory limit, it can be killed by the kernel.

### Screenshot

Add a screenshot here showing:

```text
Reason: OOMKilled
Exit Code: 137
```

> **Screenshot:** `screenshots/oomkilled.png`

---

# Task 3: Pending Pod – Requesting Too Much

## Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-hungry-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      resources:
        requests:
          cpu: "100"
          memory: "128Gi"
```

Apply:

```bash
kubectl apply -f pending-pod.yaml
```

Check:

```bash
kubectl get pod resource-hungry-pod
```

Expected:

```text
STATUS
Pending
```

Inspect:

```bash
kubectl describe pod resource-hungry-pod
```

Look at the **Events** section.

The scheduler reports an event indicating that the node does not have enough resources, for example:

```text
FailedScheduling
0/1 nodes are available: 1 Insufficient cpu.
```

or, depending on the cluster:

```text
FailedScheduling
0/1 nodes are available: 1 Insufficient memory.
```

The exact message depends on which requested resource cannot be satisfied.

### Verification

**What event message does the scheduler produce?**

**Answer:**

```text
FailedScheduling
```

with a reason such as:

```text
Insufficient cpu
```

or:

```text
Insufficient memory
```

### Screenshot

> **Screenshot:** `screenshots/pending-pod.png`

The screenshot should show the `Pending` Pod and the `FailedScheduling` event.

---

# Task 4: Liveness Probe

A liveness probe determines whether a container is still functioning.

If the liveness probe repeatedly fails, Kubernetes restarts the container.

## Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
    - name: busybox
      image: busybox:1.36
      command:
        - /bin/sh
        - -c
        - |
          touch /tmp/healthy
          sleep 30
          rm -f /tmp/healthy
          sleep 3600
      livenessProbe:
        exec:
          command:
            - cat
            - /tmp/healthy
        periodSeconds: 5
        failureThreshold: 3
```

Apply:

```bash
kubectl apply -f liveness-pod.yaml
```

Watch:

```bash
kubectl get pod liveness-pod -w
```

After approximately 30 seconds, `/tmp/healthy` is removed.

The liveness probe begins failing.

With:

```yaml
periodSeconds: 5
failureThreshold: 3
```

Kubernetes requires three consecutive failures before restarting the container.

Check:

```bash
kubectl describe pod liveness-pod
```

You should see events similar to:

```text
Liveness probe failed
Killing container
```

Check restart count:

```bash
kubectl get pod liveness-pod
```

or:

```bash
kubectl get pod liveness-pod -o jsonpath='{.status.containerStatuses[0].restartCount}'
```

### Verification

**How many times has the container restarted?**

The exact number depends on how long the Pod has been running and when it is checked.

After the first liveness failure cycle, the restart count should increase from:

```text
0
```

to:

```text
1
```

and can continue increasing because the container repeats the same behavior after each restart.

---

# Task 5: Readiness Probe

A readiness probe determines whether a Pod is ready to receive traffic.

Unlike a liveness probe:

**A readiness failure does NOT restart the container.**

Instead, Kubernetes removes the Pod from the Service's ready endpoints.

## Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      readinessProbe:
        httpGet:
          path: /
          port: 80
        periodSeconds: 5
        failureThreshold: 3
```

Apply:

```bash
kubectl apply -f readiness-pod.yaml
```

Expose it:

```bash
kubectl expose pod readiness-pod \
  --port=80 \
  --name=readiness-svc
```

Check:

```bash
kubectl get pod readiness-pod
```

Expected:

```text
READY   STATUS
1/1     Running
```

Check endpoints:

```bash
kubectl get endpoints readiness-svc
```

The Pod IP should be listed.

## Break the Readiness Probe

Delete nginx's default index page:

```bash
kubectl exec readiness-pod -- \
  rm /usr/share/nginx/html/index.html
```

Wait for the readiness probe to fail.

Check:

```bash
kubectl get pod readiness-pod
```

Expected:

```text
READY
0/1
```

Check endpoints:

```bash
kubectl get endpoints readiness-svc
```

The Pod should no longer appear as a ready endpoint.

However:

```bash
kubectl get pod readiness-pod
```

still shows the container running.

Check restart count:

```bash
kubectl get pod readiness-pod \
  -o jsonpath='{.status.containerStatuses[0].restartCount}'
```

The restart count should remain unchanged.

### Verification

**When readiness failed, was the container restarted?**

**No.**

The Pod was removed from the Service's ready endpoints, but the container was not restarted.

### Key Difference

```text
Readiness failure
       ↓
Pod is NOT ready
       ↓
Removed from Service endpoints
       ↓
Container continues running
```

---

# Task 6: Startup Probe

A startup probe is useful for containers that need significant time to initialize.

While the startup probe is still running successfully:

* Liveness probes are not used yet.
* Readiness probes are not used yet.

## Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-pod
spec:
  containers:
    - name: busybox
      image: busybox:1.36
      command:
        - /bin/sh
        - -c
        - |
          sleep 20
          touch /tmp/started
          sleep 3600

      startupProbe:
        exec:
          command:
            - cat
            - /tmp/started
        periodSeconds: 5
        failureThreshold: 12

      livenessProbe:
        exec:
          command:
            - cat
            - /tmp/started
        periodSeconds: 5
        failureThreshold: 3
```

Apply:

```bash
kubectl apply -f startup-pod.yaml
```

Watch:

```bash
kubectl get pod startup-pod -w
```

Inspect:

```bash
kubectl describe pod startup-pod
```

The container takes approximately 20 seconds to create:

```text
/tmp/started
```

The startup probe gives it a maximum budget of:

```text
12 failures × 5 seconds
= 60 seconds
```

Therefore, the container has enough time to initialize.

Once the startup probe succeeds, Kubernetes begins using the liveness probe.

## What if failureThreshold is 2?

With:

```yaml
periodSeconds: 5
failureThreshold: 2
```

the startup probe would allow approximately:

```text
2 × 5 = 10 seconds
```

before Kubernetes considers startup to have failed.

Since the application needs approximately 20 seconds to create `/tmp/started`, the startup probe would fail before initialization completes.

Kubernetes would then kill/restart the container.

### Verification

**What would happen if `failureThreshold` were 2 instead of 12?**

The startup budget would be reduced from approximately **60 seconds to 10 seconds**. Since the application needs around 20 seconds, Kubernetes would kill/restart it before it successfully starts.

---

# Probe Comparison

| Probe     | Purpose                                              | On Failure                            |
| --------- | ---------------------------------------------------- | ------------------------------------- |
| Startup   | Determines whether application has finished starting | Container can be killed/restarted     |
| Liveness  | Determines whether running container is healthy      | Container is restarted                |
| Readiness | Determines whether Pod can receive traffic           | Pod is removed from Service endpoints |

## Simple Mental Model

```text
             Container starts
                    │
                    ▼
             Startup Probe
                    │
             ┌──────┴──────┐
             │             │
          Failure        Success
             │             │
             ▼             ▼
          Restart     Liveness + Readiness
                           │
                    ┌──────┴──────┐
                    │             │
              Liveness       Readiness
                  │               │
               Failure         Failure
                  │               │
                  ▼               ▼
               Restart       Remove from
                              endpoints
```

---

# Important Probe Configuration

## periodSeconds

Controls how frequently Kubernetes performs the probe.

Example:

```yaml
periodSeconds: 5
```

means Kubernetes checks approximately every 5 seconds.

## failureThreshold

Number of consecutive failures required before Kubernetes takes action.

Example:

```yaml
failureThreshold: 3
```

means three consecutive failures are required.

## initialDelaySeconds

Delays the first probe check.

Example:

```yaml
initialDelaySeconds: 10
```

means the first probe is delayed by approximately 10 seconds.

---

# Resource Requests vs Limits

| Resource | Request                         | Limit                                              |
| -------- | ------------------------------- | -------------------------------------------------- |
| CPU      | Used by scheduler for placement | Maximum CPU usage; CPU can be throttled            |
| Memory   | Used by scheduler for placement | Maximum memory; exceeding it can result in OOMKill |

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
```

Here:

```text
CPU request  = 100m = 0.1 CPU
CPU limit    = 250m = 0.25 CPU

Memory request = 128Mi
Memory limit   = 256Mi
```

---

# QoS Class Summary

### Guaranteed

Requests and limits are configured equally for every container.

```yaml
requests:
  cpu: "500m"
  memory: "512Mi"
limits:
  cpu: "500m"
  memory: "512Mi"
```

### Burstable

Requests and/or limits are configured, but the Pod does not meet the Guaranteed requirements.

Example:

```yaml
requests:
  cpu: "100m"
  memory: "128Mi"
limits:
  cpu: "250m"
  memory: "256Mi"
```

### BestEffort

No CPU or memory requests/limits are configured.

---

# Screenshots

The following screenshots should be added to this documentation.

## 1. OOMKilled

Show:

```text
Reason: OOMKilled
Exit Code: 137
```

> Add screenshot: `screenshots/oomkilled.png`

## 2. Pending Pod

Show:

```text
STATUS: Pending
```

and the scheduler event:

```text
FailedScheduling
Insufficient cpu
```

or:

```text
FailedScheduling
Insufficient memory
```

> Add screenshot: `screenshots/pending-pod.png`

## 3. Liveness Probe

Show the event:

```text
Liveness probe failed
```

and the increasing restart count.

> Add screenshot: `screenshots/liveness-probe.png`

## 4. Readiness Probe

Show:

```text
READY: 0/1
```

and the empty Service endpoints.

> Add screenshot: `screenshots/readiness-probe.png`

## 5. Startup Probe

Show the startup probe events and successful transition to the running state.

> Add screenshot: `screenshots/startup-probe.png`

---

# Useful Verification Commands

```bash
kubectl get pods
```

```bash
kubectl describe pod <pod-name>
```

```bash
kubectl get pod <pod-name> -o yaml
```

Check QoS:

```bash
kubectl get pod <pod-name> \
  -o jsonpath='{.status.qosClass}'
```

Check restart count:

```bash
kubectl get pod <pod-name> \
  -o jsonpath='{.status.containerStatuses[0].restartCount}'
```

Check Service endpoints:

```bash
kubectl get endpoints <service-name>
```

Watch Pod changes:

```bash
kubectl get pods -w
```

View recent events:

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

# Task 7: Clean Up

Delete the Pods:

```bash
kubectl delete pod resource-pod
kubectl delete pod oom-pod
kubectl delete pod resource-hungry-pod
kubectl delete pod liveness-pod
kubectl delete pod readiness-pod
kubectl delete pod startup-pod
```

Delete the Service:

```bash
kubectl delete service readiness-svc
```

Verify:

```bash
kubectl get pods
kubectl get services
```

---

# Final Learnings

Today I learned that Kubernetes uses resource requests to make scheduling decisions and resource limits to control container resource consumption.

I also observed that exceeding a memory limit can result in an `OOMKilled` container with exit code `137`, while CPU can be throttled when its limit is reached.

The three probes have different responsibilities:

* **Startup probe** protects slow-starting applications.
* **Liveness probe** detects containers that are no longer healthy and restarts them.
* **Readiness probe** controls whether a Pod receives Service traffic without restarting the container.

These mechanisms allow Kubernetes to make better scheduling decisions and automatically respond to unhealthy workloads.

---

# Key Takeaways

```text
Requests → Scheduling
Limits   → Runtime enforcement

CPU      → Can be throttled
Memory   → Can trigger OOMKill

Liveness  → Restart
Readiness → Remove from endpoints
Startup   → Protect slow startup

OOMKilled → Exit Code 137

QoS:
Guaranteed → Requests == Limits
Burstable  → Partial/different resources
BestEffort → No resources configured
```

**Day 57 completed: Kubernetes Resource Management and Probes.**