# Day 56 – Kubernetes StatefulSets

## Overview

Today I learned about **Kubernetes StatefulSets**, a workload controller designed for stateful applications that require:

* Stable and predictable pod names
* Ordered pod creation and termination
* Stable network identities
* Persistent storage for each replica
* Data persistence across pod recreation

StatefulSets are commonly used for applications such as **MySQL, PostgreSQL, MongoDB, Kafka, and other distributed/stateful systems**.

---

# Task 1 – Understand the Problem

## Deployment

First, I created a Deployment with 3 replicas using NGINX:

```bash
kubectl create deployment nginx-deployment --image=nginx --replicas=3
```

Check the pods:

```bash
kubectl get pods
```

The pod names are generated with random suffixes, for example:

```text
nginx-deployment-7d8b495c8f-x7k2m
nginx-deployment-7d8b495c8f-p4z9q
nginx-deployment-7d8b495c8f-r6k8v
```

If one pod is deleted:

```bash
kubectl delete pod <pod-name>
```

Kubernetes creates a replacement pod with a different generated name.

### Why is this a problem for databases?

Random pod names are not ideal for database clusters because database nodes often need a **stable identity**.

For example, a database cluster may need to know:

```text
database-0
database-1
database-2
```

so that each node can maintain its identity, storage, and cluster membership.

A Deployment does not provide this stable identity.

### Deployment vs StatefulSet

| Feature          | Deployment                        | StatefulSet           |
| ---------------- | --------------------------------- | --------------------- |
| Pod names        | Random/generated                  | Stable and ordered    |
| Example          | `app-x7k2m`                       | `app-0`               |
| Startup          | Generally parallel                | Ordered by default    |
| Storage          | Usually shared/externally managed | Per-pod PVC           |
| Network identity | No stable pod DNS                 | Stable pod DNS        |
| Best suited for  | Stateless applications            | Stateful applications |

After completing this task, I deleted the Deployment:

```bash
kubectl delete deployment nginx-deployment
```

---

# Task 2 – Create a Headless Service

A StatefulSet uses a **Headless Service** to provide stable DNS records for individual pods.

I created the following Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
  namespace: nginx
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

Apply it:

```bash
kubectl apply -f headless-service.yaml
```

Verify:

```bash
kubectl get svc -n nginx
```

Expected output:

```text
NAME             TYPE        CLUSTER-IP   PORT(S)
nginx-headless   ClusterIP   None         80/TCP
```

### Verification

The `CLUSTER-IP` column shows:

```text
None
```

This confirms that the Service is **Headless**.

Unlike a normal Service, a Headless Service does not provide a single virtual IP for load balancing. Instead, DNS can resolve individual StatefulSet pods.

---

# Task 3 – Create a StatefulSet

I created a StatefulSet with:

* 3 replicas
* NGINX image
* Stable pod names
* 100Mi persistent storage per pod
* ReadWriteOnce access mode

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
  namespace: nginx
spec:
  serviceName: nginx-headless
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
          image: nginx:latest
          ports:
            - containerPort: 80
              name: http

          volumeMounts:
            - name: nginx-storage
              mountPath: /usr/share/nginx/html

  volumeClaimTemplates:
    - metadata:
        name: nginx-storage

      spec:
        accessModes:
          - ReadWriteOnce

        resources:
          requests:
            storage: 100Mi
```

Apply the StatefulSet:

```bash
kubectl apply -f statefulset.yaml
```

Watch the pods:

```bash
kubectl get pods -n nginx -w
```

The StatefulSet creates the pods in order:

```text
nginx-statefulset-0
nginx-statefulset-1
nginx-statefulset-2
```

Pod `1` is created after pod `0` becomes Ready, and pod `2` is created after pod `1` becomes Ready.

### Verify StatefulSet

```bash
kubectl get sts -n nginx
```

Expected:

```text
NAME                READY
nginx-statefulset   3/3
```

### Verify Pods

```bash
kubectl get pods -n nginx
```

Expected:

```text
NAME                  READY   STATUS
nginx-statefulset-0   1/1     Running
nginx-statefulset-1   1/1     Running
nginx-statefulset-2   1/1     Running
```

### Verify PVCs

```bash
kubectl get pvc -n nginx
```

Expected PVC names:

```text
nginx-storage-nginx-statefulset-0
nginx-storage-nginx-statefulset-1
nginx-storage-nginx-statefulset-2
```

The naming pattern is:

```text
<volumeClaimTemplate>-<statefulset-name>-<ordinal>
```

Therefore:

```text
nginx-storage-nginx-statefulset-0
nginx-storage-nginx-statefulset-1
nginx-storage-nginx-statefulset-2
```

### Verification

The exact pod names are:

```text
nginx-statefulset-0
nginx-statefulset-1
nginx-statefulset-2
```

Each pod receives its own PVC.

---

# Task 4 – Stable Network Identity

Each StatefulSet pod receives a predictable DNS name.

The general format is:

```text
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

For my StatefulSet:

```text
nginx-statefulset-0.nginx-headless.nginx.svc.cluster.local
nginx-statefulset-1.nginx-headless.nginx.svc.cluster.local
nginx-statefulset-2.nginx-headless.nginx.svc.cluster.local
```

I started a temporary BusyBox pod:

```bash
kubectl run busybox \
  --image=busybox:1.36 \
  --restart=Never \
  -it --rm \
  -n nginx \
  -- sh
```

Inside the BusyBox container:

```bash
nslookup nginx-statefulset-0.nginx-headless.nginx.svc.cluster.local
```

Then:

```bash
nslookup nginx-statefulset-1.nginx-headless.nginx.svc.cluster.local
```

And:

```bash
nslookup nginx-statefulset-2.nginx-headless.nginx.svc.cluster.local
```

I compared the DNS results with:

```bash
kubectl get pods -n nginx -o wide
```

The DNS-resolved IP addresses matched the individual pod IP addresses.

### Verification

Yes, the `nslookup` IP matches the corresponding StatefulSet pod IP.

This demonstrates that StatefulSets provide **stable network identity** through the Headless Service.

---

# Task 5 – Stable Storage

The main advantage of StatefulSets is that each pod gets its own persistent storage.

I wrote unique data to `nginx-statefulset-0`:

```bash
kubectl exec -n nginx nginx-statefulset-0 -- \
  sh -c "echo 'Data from nginx-statefulset-0' > /usr/share/nginx/html/index.html"
```

Verify the data:

```bash
kubectl exec -n nginx nginx-statefulset-0 -- \
  cat /usr/share/nginx/html/index.html
```

Output:

```text
Data from nginx-statefulset-0
```

## Delete the Pod

```bash
kubectl delete pod nginx-statefulset-0 -n nginx
```

The StatefulSet automatically creates the replacement:

```text
nginx-statefulset-0
```

Notice that the pod name is the **same**.

Wait until it becomes Ready:

```bash
kubectl get pods -n nginx
```

Then check the data again:

```bash
kubectl exec -n nginx nginx-statefulset-0 -- \
  cat /usr/share/nginx/html/index.html
```

Expected:

```text
Data from nginx-statefulset-0
```

### Verification

Yes. The data remains identical after pod recreation.

This happens because the replacement pod reconnects to the **same PVC**.

The pod is replaced, but the persistent volume claim remains.

---

# Task 6 – Ordered Scaling

## Scale Up

I scaled the StatefulSet from 3 replicas to 5:

```bash
kubectl scale statefulset nginx-statefulset --replicas=5 -n nginx
```

Watch the pods:

```bash
kubectl get pods -n nginx -w
```

Kubernetes creates:

```text
nginx-statefulset-3
```

followed by:

```text
nginx-statefulset-4
```

Verify:

```bash
kubectl get pods -n nginx
```

Expected:

```text
nginx-statefulset-0
nginx-statefulset-1
nginx-statefulset-2
nginx-statefulset-3
nginx-statefulset-4
```

## Scale Down

I then scaled the StatefulSet back to 3 replicas:

```bash
kubectl scale statefulset nginx-statefulset --replicas=3 -n nginx
```

StatefulSets terminate pods in reverse order:

```text
nginx-statefulset-4
nginx-statefulset-3
```

The lower ordinal pods remain:

```text
nginx-statefulset-0
nginx-statefulset-1
nginx-statefulset-2
```

## Verify PVCs

```bash
kubectl get pvc -n nginx
```

The PVCs for the removed replicas remain.

For example:

```text
nginx-storage-nginx-statefulset-0
nginx-storage-nginx-statefulset-1
nginx-storage-nginx-statefulset-2
nginx-storage-nginx-statefulset-3
nginx-storage-nginx-statefulset-4
```

### Verification

After scaling down from 5 to 3, **5 PVCs still exist**.

The PVCs are not automatically deleted during StatefulSet scale-down, which helps preserve data.

---

# Task 7 – Clean Up

First, I deleted the StatefulSet:

```bash
kubectl delete statefulset nginx-statefulset -n nginx
```

Then deleted the Headless Service:

```bash
kubectl delete service nginx-headless -n nginx
```

I checked the PVCs:

```bash
kubectl get pvc -n nginx
```

The PVCs were still present.

This demonstrates that deleting a StatefulSet does not automatically delete its PVCs.

## Delete PVCs Manually

To remove the PVCs:

```bash
kubectl delete pvc --all -n nginx
```

Verify:

```bash
kubectl get pvc -n nginx
```

Expected:

```text
No resources found in nginx namespace.
```

---

# Key Concepts Learned

## StatefulSet

A StatefulSet manages stateful applications where each pod requires a stable identity.

Important properties include:

* Stable pod names
* Stable network identity
* Ordered deployment
* Ordered scaling
* Persistent storage
* Stable pod-to-PVC relationships

---

## Headless Service

A Headless Service is created using:

```yaml
clusterIP: None
```

It provides DNS records for individual StatefulSet pods.

Example:

```text
nginx-statefulset-0.nginx-headless.nginx.svc.cluster.local
```

---

## Stable Pod Identity

Unlike Deployments:

```text
nginx-deployment-7d8b495c8f-x7k2m
```

StatefulSets provide:

```text
nginx-statefulset-0
nginx-statefulset-1
nginx-statefulset-2
```

If `nginx-statefulset-1` is deleted, Kubernetes recreates:

```text
nginx-statefulset-1
```

rather than generating a completely new identity.

---

## volumeClaimTemplates

`volumeClaimTemplates` automatically creates an individual PVC for each StatefulSet replica.

For example:

```text
nginx-storage-nginx-statefulset-0
nginx-storage-nginx-statefulset-1
nginx-storage-nginx-statefulset-2
```

Each pod gets its own storage.

---

# Deployment vs StatefulSet

| Feature          | Deployment                   | StatefulSet              |
| ---------------- | ---------------------------- | ------------------------ |
| Pod identity     | Random/generated             | Stable                   |
| Pod naming       | `app-x7k2m`                  | `app-0`                  |
| Ordering         | Usually parallel             | Ordered                  |
| Network identity | Dynamic                      | Stable                   |
| DNS per pod      | No stable identity           | Yes                      |
| Storage          | Usually external/shared      | Per-pod PVC              |
| Scale-up         | Pods can start independently | Ordered                  |
| Scale-down       | No guaranteed identity order | Reverse order            |
| Typical use      | Stateless applications       | Stateful applications    |
| Examples         | NGINX, frontend, API         | MySQL, PostgreSQL, Kafka |

---

# Important Commands

### List StatefulSets

```bash
kubectl get sts -n nginx
```

### Describe StatefulSet

```bash
kubectl describe sts nginx-statefulset -n nginx
```

### List StatefulSet Pods

```bash
kubectl get pods -n nginx
```

### List PVCs

```bash
kubectl get pvc -n nginx
```

### Scale StatefulSet

```bash
kubectl scale sts nginx-statefulset --replicas=5 -n nginx
```

### Watch Pods

```bash
kubectl get pods -n nginx -w
```

### Check Pod IPs

```bash
kubectl get pods -n nginx -o wide
```

### Delete a Pod

```bash
kubectl delete pod nginx-statefulset-0 -n nginx
```

### Delete StatefulSet

```bash
kubectl delete sts nginx-statefulset -n nginx
```

### Delete PVCs

```bash
kubectl delete pvc --all -n nginx
```

---

# Screenshots

The following screenshots should be added to this documentation as evidence of the practical work.

## 1. StatefulSet Pods

Screenshot showing:

```bash
kubectl get pods -n nginx
```

Expected:

```text
nginx-statefulset-0
nginx-statefulset-1
nginx-statefulset-2
```

**Screenshot:** `screenshots/statefulset-pods.png`

---

## 2. StatefulSet and PVCs

Screenshot showing:

```bash
kubectl get sts -n nginx
kubectl get pvc -n nginx
```

**Screenshot:** `screenshots/statefulset-pvc.png`

---

## 3. Headless Service

Screenshot showing:

```bash
kubectl get svc -n nginx
```

Expected:

```text
CLUSTER-IP
None
```

**Screenshot:** `screenshots/headless-service.png`

---

## 4. DNS Resolution

Screenshot showing:

```bash
nslookup nginx-statefulset-0.nginx-headless.nginx.svc.cluster.local
```

and the corresponding:

```bash
kubectl get pods -n nginx -o wide
```

**Screenshot:** `screenshots/statefulset-dns.png`

---

## 5. Data Persistence

Screenshot showing the data before and after deleting the pod:

```bash
kubectl exec -n nginx nginx-statefulset-0 -- \
  cat /usr/share/nginx/html/index.html
```

Expected:

```text
Data from nginx-statefulset-0
```

**Screenshot:** `screenshots/data-persistence.png`

---

# Final Verification

| Requirement                              | Result    |
| ---------------------------------------- | --------- |
| StatefulSet with 3 replicas              | Completed |
| Stable pod names                         | Verified  |
| Ordered pod creation                     | Verified  |
| Headless Service                         | Verified  |
| `CLUSTER-IP: None`                       | Verified  |
| Individual pod DNS                       | Verified  |
| Per-pod PVCs                             | Verified  |
| Data survives pod deletion               | Verified  |
| Ordered scale-up                         | Verified  |
| Reverse-order scale-down                 | Verified  |
| PVCs retained after scale-down           | Verified  |
| PVCs retained after StatefulSet deletion | Verified  |
| Manual PVC cleanup                       | Completed |

---

# What I Learned

Today I learned why Kubernetes StatefulSets are important for stateful applications.

A Deployment is ideal for stateless workloads where individual pod identity does not matter. StatefulSets are designed for applications where each replica needs a **stable identity, stable network address, and persistent storage**.

The key concepts I learned were:

```text
StatefulSet
    ↓
Stable Pod Identity
    ↓
Stable DNS
    ↓
Headless Service
    ↓
Persistent Storage
    ↓
volumeClaimTemplates
```

The most important takeaway is that deleting a StatefulSet pod does not mean losing its identity or data. Kubernetes recreates the same ordinal pod and reconnects it to its persistent storage.

---

# Conclusion

StatefulSets provide the Kubernetes primitives required by many stateful and distributed applications.

They provide:

* Stable pod names
* Ordered creation
* Ordered termination
* Stable DNS
* Per-pod persistent storage
* Data persistence across pod recreation

This makes StatefulSets an important Kubernetes workload for stateful applications such as databases and distributed systems.
