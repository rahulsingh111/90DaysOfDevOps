### Task 1: Create Your First Pod (Nginx)

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```
yes I can see the nginx welcome page when I curl from inside the pod

### Task 2: Create a Custom Pod (BusyBox)
```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  labels:
    app: busybox
    environment: dev
spec:
  containers:
    - name: busybox
      image: busybox:latest
      command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
```

### Task 3: Imperative vs Declarative
```
kubectl get pod redis-pod -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-09-06T16:08:07Z"
  generation: 1
  labels:
    run: redis-pod
  name: redis-pod
  namespace: default
  resourceVersion: "1078"
  uid: 2bdc6970-4b46-4a25-b15f-a0a0d0c0aa84
spec:
  containers:
  - image: redis:latest
    imagePullPolicy: Always
    name: redis-pod
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-px5x2
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-px5x2
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2026-09-06T16:08:07Z"
    observedGeneration: 1
    status: "False"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2026-09-06T16:08:07Z"
    observedGeneration: 1
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-09-06T16:08:07Z"
    message: 'containers with unready status: [redis-pod]'
    observedGeneration: 1
    reason: ContainersNotReady
    status: "False"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-09-06T16:08:07Z"
    message: 'containers with unready status: [redis-pod]'
    observedGeneration: 1
    reason: ContainersNotReady
    status: "False"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-09-06T16:08:07Z"
    observedGeneration: 1
    status: "True"
    type: PodScheduled
  containerStatuses:
  - image: redis:latest
    imageID: ""
    lastState: {}
    name: redis-pod
    ready: false
    restartCount: 0
    started: false
    state:
      waiting:
        reason: ContainerCreating
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-px5x2
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 192.168.49.2
  hostIPs:
  - ip: 192.168.49.2
  observedGeneration: 1
  phase: Pending
  qosClass: BestEffort
  startTime: "2026-09-06T16:08:07Z"

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl run test-pod --image=nginx --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test-pod
  name: test-pod
spec:
  containers:
  - image: nginx
    name: test-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -o wide
NAME        READY   STATUS              RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
busybox     1/1     Running             0          7m8s   10.244.0.4   minikube   <none>           <none>
nginx-pod   1/1     Running             0          14m    10.244.0.3   minikube   <none>           <none>
redis-pod   0/1     ContainerCreating   0          72s    <none>       minikube   <none>           <none>
```

### Task 4: Validate Before Applying

```
Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl apply -f nginx-pod.yaml --dry-run=server
The Pod "nginx-pod" is invalid: spec.containers[0].image: Required value

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl apply -f nginx-pod.yaml --dry-run=client
pod/nginx-pod configured (dry run)
```

### Task 5: Pod Labels and Filtering

```
$ kubectl logs linux-pod
Hello from Ubuntu

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
busybox     1/1     Running   0          22m   10.244.0.4   minikube   <none>           <none>
linux-pod   1/1     Running   0          93s   10.244.0.7   minikube   <none>           <none>
nginx-pod   1/1     Running   0          30m   10.244.0.3   minikube   <none>           <none>
redis-pod   1/1     Running   0          16m   10.244.0.5   minikube   <none>           <none>

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods --show-labels
NAME        READY   STATUS    RESTARTS   AGE    LABELS
busybox     1/1     Running   0          23m    app=busybox,environment=dev
linux-pod   1/1     Running   0          2m9s   app=linux,environment=production,team=devops
nginx-pod   1/1     Running   0          30m    app=nginx
redis-pod   1/1     Running   0          17m    run=redis-pod
```

### Task 6: Clean Up

```
$ kubectl delete pod nginx-pod
pod "nginx-pod" deleted from default namespace

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl delete pod busybox-pod
Error from server (NotFound): pods "busybox-pod" not found

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl delete pod busybox
pod "busybox" deleted from default namespace

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl delete pod redis-pod
pod "redis-pod" deleted from default namespace

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl delete -f linux-pod.yaml 
pod "linux-pod" deleted from default namespace

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods
No resources found in default namespace.
```