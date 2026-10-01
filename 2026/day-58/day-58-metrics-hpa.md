# Day 58 – Metrics Server and Horizontal Pod Autoscaler (HPA)

## Overview

Today I worked with **Kubernetes Metrics Server and Horizontal Pod Autoscaler (HPA)**.

The goal was to make Kubernetes monitor actual CPU usage and automatically scale an application based on CPU utilization.

In this lab I:

* Used CPU resource requests for the application.
* Used Metrics Server to collect resource usage.
* Created an HPA targeting 50% CPU utilization.
* Generated artificial traffic against the application.
* Observed HPA automatically scale the application from **1 replica to 9 replicas**.
* Configured declarative HPA behavior using `autoscaling/v2`.
* Tested scale-down behavior with a 300-second stabilization window.

---

# Task 1 – Metrics Server

Metrics Server provides resource usage metrics such as CPU and memory to Kubernetes.

HPA depends on these metrics to determine whether more or fewer replicas are required.

The important distinction is:

```text
kubectl top
    ↓
Metrics Server
    ↓
Actual CPU / Memory usage
    ↓
HPA
    ↓
Desired number of replicas
```

Metrics Server reports actual resource consumption. It is different from the CPU and memory requests/limits configured in a Pod.

### Verify Metrics Server

```bash
kubectl get pods -n kube-system | grep metrics-server
```

Check node metrics:

```bash
kubectl top nodes
```

Check all Pod metrics:

```bash
kubectl top pods -A
```

---

# Task 2 – Exploring kubectl top

The `kubectl top` command shows the current resource consumption reported by Metrics Server.

For example:

```bash
kubectl top pods -n hpa
```

During my HPA test, I observed:

```text
NAME                              CPU(cores)   MEMORY(bytes)
hpa-deployment-69b84d7f86-86bjp   1m           8Mi
```

This demonstrated that Metrics Server was successfully providing CPU and memory data.

### Useful commands

```bash
kubectl top nodes
```

```bash
kubectl top pods -A
```

```bash
kubectl top pods -A --sort-by=cpu
```

### Important distinction

`kubectl top` shows **actual resource usage**.

It does not show the resource requests or limits configured on the container.

To inspect requests and limits:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

---

# Task 3 – Deployment with CPU Requests

I created a Deployment using the CPU-intensive PHP-Apache example image:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-deployment
  namespace: hpa
  labels:
    app: hpa
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa
  template:
    metadata:
      labels:
        app: hpa
    spec:
      containers:
        - name: hpa-example
          image: registry.k8s.io/hpa-example
          resources:
            requests:
              cpu: "200m"
```

### Why `200m`?

The CPU request is important because HPA CPU utilization is calculated relative to the requested CPU.

For example:

```text
CPU request = 200m
CPU usage   = 100m

Utilization = 100m / 200m × 100
            = 50%
```

Without a CPU request, CPU utilization-based HPA cannot calculate the percentage correctly.

---

# Task 4 – Expose the Application

The Deployment was exposed through a ClusterIP Service:

```bash
kubectl expose deployment hpa-deployment --port=80 -n hpa
```

The Service provides a stable DNS name:

```text
hpa-deployment
```

The load generator can therefore access the application using:

```text
http://hpa-deployment
```

---

# Task 5 – Imperative HPA

The initial HPA was created using the imperative command:

```bash
kubectl autoscale deployment hpa-deployment \
  --cpu-percent=50 \
  --min=1 \
  --max=10 \
  -n hpa
```

The newer Kubernetes CLI reports that `--cpu-percent` is deprecated and recommends the newer `--cpu` syntax.

The HPA configuration was:

```text
Minimum replicas: 1
Maximum replicas: 10
CPU target: 50%
```

Check the HPA:

```bash
kubectl get hpa -n hpa
```

---

# Task 6 – Generate Load

To generate CPU load, I created a BusyBox load-generator:

```bash
MSYS_NO_PATHCONV=1 kubectl run load-generator \
  --image=busybox:1.36 \
  --restart=Never \
  -n hpa \
  -- /bin/sh -c "while true; do wget -q -O- http://hpa-deployment; done"
```

## Git Bash / Windows Issue

Initially, the container failed to start.

The problem was caused by Git Bash converting:

```text
/bin/sh
```

into:

```text
C:/Program Files/Git/usr/bin/sh
```

The Kubernetes container therefore tried to execute a Windows path inside the Linux BusyBox container.

The error was:

```text
exec: "C:/Program Files/Git/usr/bin/sh":
stat C:/Program Files/Git/usr/bin/sh:
no such file or directory
```

This resulted in:

```text
ContainerCannotRun
```

The `kubectl describe pod` output confirmed the path-conversion problem.

### Solution

I used:

```bash
MSYS_NO_PATHCONV=1
```

This prevents Git Bash from converting the Linux path.

After recreating the Pod:

```text
hpa-deployment-69b84d7f86-86bjp   1/1   Running
load-generator                    1/1   Running
```

The load generator was successfully running.

---

# Task 7 – HPA Scaling Under Load

I watched the HPA using:

```bash
kubectl get hpa -n hpa -w
```

The HPA initially showed:

```text
cpu: 0%/50%
```

After the load generator started sending continuous HTTP requests, CPU usage increased significantly.

I observed:

```text
cpu: 431%/50%
```

The HPA then scaled the Deployment:

```text
1 → 4 → 8 → 9 replicas
```

The actual terminal output showed:

```text
cpu: 431%/50%   replicas: 1
cpu: 431%/50%   replicas: 4
cpu: 431%/50%   replicas: 8
cpu: 431%/50%   replicas: 9
```

CPU utilization subsequently decreased as additional replicas handled the workload:

```text
431% → 172% → 53%
```

The final observed utilization was approximately:

```text
53% / 50%
```

## Result

The HPA successfully scaled the application from:

```text
1 replica
    ↓
4 replicas
    ↓
8 replicas
    ↓
9 replicas
```

The maximum configured replica count was 10.

At the peak of my test, **9 replicas were running**.

The Pod list confirmed nine application Pods plus the load-generator Pod.

---

# Task 8 – Stop the Load

After testing, I deleted the load-generator:

```bash
kubectl delete pod load-generator -n hpa
```

The load stopped, but the Deployment remained at 9 replicas initially.

This is expected because the HPA was configured with a scale-down stabilization window.

---

# Task 9 – Declarative HPA with autoscaling/v2

I removed the imperative HPA:

```bash
kubectl delete hpa hpa-deployment -n hpa
```

Then I created an HPA manifest using:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
```

## HPA Manifest

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: deployment-hpa
  namespace: hpa

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-deployment

  minReplicas: 1
  maxReplicas: 10

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15

    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
```

The first version of the YAML contained indentation and field-name mistakes. Kubernetes rejected it with errors involving:

```text
spec.scaleTargetRef.apiversion
spec.scaleTargetRef.minReplicas
spec.scaleTargetRef.maxReplicas
spec.scaleTargetRef.metrics
spec.scaleTargetRef.behaviour
```

After correcting the manifest, it was successfully created:

```text
horizontalpodautoscaler.autoscaling/deployment-hpa created
```

---

# Understanding HPA Behavior

The `behavior` section controls how quickly the HPA is allowed to scale up or scale down.

## Scale Up

```yaml
scaleUp:
  stabilizationWindowSeconds: 0
```

This means there is no stabilization delay before scaling up.

The policy:

```yaml
- type: Percent
  value: 100
  periodSeconds: 15
```

allows the HPA to increase the replica count by up to 100% within a 15-second period, subject to the HPA's calculated desired replicas and maximum replica limit.

---

## Scale Down

```yaml
scaleDown:
  stabilizationWindowSeconds: 300
```

300 seconds equals:

```text
5 minutes
```

This prevents the HPA from immediately removing Pods when CPU utilization temporarily drops.

It provides a stabilization period before reducing replicas.

---

# HPA Formula

The basic HPA calculation can be represented as:

```text
desiredReplicas =
ceil(currentReplicas × currentUsage / targetUsage)
```

For example, if:

```text
Current replicas = 2
Current CPU       = 100%
Target CPU        = 50%
```

then:

```text
desiredReplicas =
ceil(2 × 100 / 50)

= 4
```

Therefore, HPA may increase the Deployment toward 4 replicas, subject to its configured limits and scaling behavior.

---

# autoscaling/v1 vs autoscaling/v2

## autoscaling/v1

The original HPA API is primarily focused on CPU utilization.

Example:

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
```

It is useful for basic CPU-based autoscaling.

## autoscaling/v2

`autoscaling/v2` provides more advanced HPA functionality.

It supports:

* CPU metrics
* Memory metrics
* Multiple metrics
* More detailed scaling behavior
* Scale-up policies
* Scale-down policies
* Stabilization windows

Example:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
```

For more advanced HPA configurations, `autoscaling/v2` provides much finer control.

---

# Useful Commands

### Check HPA

```bash
kubectl get hpa -n hpa
```

### Watch HPA

```bash
kubectl get hpa -n hpa -w
```

### Describe HPA

```bash
kubectl describe hpa deployment-hpa -n hpa
```

### Check Deployment

```bash
kubectl get deployment hpa-deployment -n hpa
```

### Watch Pods

```bash
kubectl get pods -n hpa -w
```

### Check CPU and memory

```bash
kubectl top pods -n hpa
```

### Check node usage

```bash
kubectl top nodes
```

---

# Key Learnings

### 1. Metrics Server

Metrics Server provides the resource usage data required by HPA.

### 2. CPU Requests Matter

HPA CPU utilization is calculated relative to the CPU request.

```yaml
resources:
  requests:
    cpu: "200m"
```

### 3. `kubectl top` Shows Actual Usage

```bash
kubectl top pods
```

shows actual CPU and memory consumption, not configured requests or limits.

### 4. HPA Automatically Changes Replicas

Instead of manually changing:

```yaml
replicas:
```

HPA manages the desired number of replicas based on observed metrics.

### 5. Scale-Up and Scale-Down Behave Differently

Fast scale-up and stabilized scale-down can prevent an application from reacting too slowly to traffic increases while avoiding unnecessary Pod churn during temporary traffic drops.

### 6. Windows Git Bash Can Affect Kubernetes Commands

When using Git Bash with Linux containers, Linux paths such as:

```text
/bin/sh
```

can be incorrectly converted to Windows paths.

Using:

```bash
MSYS_NO_PATHCONV=1
```

prevents this conversion.

---

# Final Result

The HPA successfully demonstrated automatic horizontal scaling.

Under load:

```text
1 replica
   ↓
4 replicas
   ↓
8 replicas
   ↓
9 replicas
```

CPU utilization initially reached:

```text
431%
```

against a target of:

```text
50%
```

As replicas increased, utilization dropped to approximately:

```text
53%
```

This demonstrated Kubernetes automatically distributing the workload across additional Pods.

---

# Cleanup

After completing the experiment, the application resources can be removed with:

```bash
kubectl delete hpa deployment-hpa -n hpa
kubectl delete service hpa-deployment -n hpa
kubectl delete deployment hpa-deployment -n hpa
kubectl delete pod load-generator -n hpa
```

Metrics Server should remain installed.

---

# Day 58 Summary

Today I learned how Kubernetes uses **Metrics Server + HPA** to automatically scale workloads based on real resource usage.

The most important concepts were:

```text
Resource Requests
       ↓
Metrics Server
       ↓
Actual CPU Usage
       ↓
HPA
       ↓
Desired Replicas
       ↓
Deployment Scaling
```

The most practical part of the exercise was watching the application scale from **1 to 9 replicas under CPU load**.

This demonstrates the basic mechanism behind horizontal scaling in Kubernetes and provides a foundation for working with more advanced autoscaling strategies.