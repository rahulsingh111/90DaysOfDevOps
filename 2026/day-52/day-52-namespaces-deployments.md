Namespaces are used to distinguish between different environments or teams within the same cluster. For example, you might have a `dev` namespace for development, a `staging` namespace for testing, and a `production` namespace for live applications. These are used to segregate resources and manage access control.

Deployment manifest
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
spec:
  replicas: 4
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
        image: nginx:1.24
        ports:
        - containerPort: 80
```
apiVersion is used to specify the version of the Kubernetes API that you are using. In this case, we are using `apps/v1` which is the stable version for Deployments.
kind is used to specify the type of resource that you are creating. In this case, we are creating a Deployment.
metadata is used to specify metadata about the resource, such as its name and namespace. In this case, we are creating a Deployment named `nginx-deployment` in the `dev` namespace.
labels are used to specify key-value pairs that can be used to identify and organize resources. In this case, we are labeling the Deployment with `app: nginx`.
spec is used to specify the desired state of the resource. In this case, we are specifying that we want 4 replicas of the Pod running at all times,matchLabels are used to specify the labels that should be applied to the Pods created by the Deployment. In this case, we are matching Pods with the label `app: nginx`. template is used to specify the template for the Pods that will be created by the Deployment. In this case, we are specifying that the Pods should have the label `app: nginx` and should run a container named `nginx` using the `nginx:1.24` image, and expose port 80.

When we delete a pod created by a Deployment, the Deployment controller will automatically create a new pod to replace it. This ensures that the desired number of replicas is always maintained. On the other hand if we remove a standalone pod, it will not be recreated automatically. This is because the Deployment controller is responsible for managing the lifecycle of the pods created by the Deployment, while standalone pods are not managed by any controller.

Scaling is used to increase or decrease the number of replicas of a Deployment. This can be done using the `kubectl scale` command. Imperative scaling is when you specify the desired number of replicas directly, while declarative scaling is when you update the Deployment manifest to specify the desired number of replicas and then apply the changes using `kubectl apply`. While declarative scaling works when we update the Deployment manifest, imperative scaling is more suitable for temporary changes or quick adjustments.

Rolling updates are used to update the image of a Deployment. This can be done using the `kubectl set image` command. Imperative rolling updates are when you specify the new image directly, while declarative rolling updates are when you update the Deployment manifest to specify the new image and then apply the changes using `kubectl apply`. While declarative rolling updates work when we update the Deployment manifest, imperative rolling updates are more suitable for temporary changes or quick adjustments. Rollbacks are used to revert a Deployment to a previous version. This can be done using the `kubectl rollout undo` command. Imperative rollbacks are when you specify the previous version directly, while declarative rollbacks are when you update the Deployment manifest to specify the previous version and then apply the changes using `kubectl apply`. While declarative rollbacks work when we update the Deployment manifest, imperative rollbacks are more suitable for temporary changes or quick adjustments.

screenshot of the output
```
kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get deployments -n dev
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3     3            0           7s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-h6xcf   0/1     ContainerCreating   0          15s
nginx-deployment-6d8cd7bf4b-vf6cr   0/1     ContainerCreating   0          15s
nginx-deployment-6d8cd7bf4b-xgxz9   0/1     ContainerCreating   0          15s
nginx-dev                           1/1     Running             0          8m10s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-h6xcf   1/1     Running             0          31s
nginx-deployment-6d8cd7bf4b-vf6cr   0/1     ContainerCreating   0          31s
nginx-deployment-6d8cd7bf4b-xgxz9   0/1     ContainerCreating   0          31s
nginx-dev                           1/1     Running             0          8m26s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-h6xcf   1/1     Running   0          91s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          91s
nginx-deployment-6d8cd7bf4b-xgxz9   1/1     Running   0          91s
nginx-dev                           1/1     Running   0          9m26s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl delete pod nginx-deployment-6d8cd7bf4b-xgxz9 -n dev
pod "nginx-deployment-6d8cd7bf4b-xgxz9" deleted from dev namespace

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-h6xcf   1/1     Running   0          2m7s
nginx-deployment-6d8cd7bf4b-qr2ds   1/1     Running   0          5s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          2m7s
nginx-dev                           1/1     Running   0          10m

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-h6xcf   1/1     Running   0          2m17s
nginx-deployment-6d8cd7bf4b-qr2ds   1/1     Running   0          15s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          2m17s
nginx-dev                           1/1     Running   0          10m

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl delete pod nginx-dev -n dev
pod "nginx-dev" deleted from dev namespace

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-h6xcf   1/1     Running   0          2m29s
nginx-deployment-6d8cd7bf4b-qr2ds   1/1     Running   0          27s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          2m29s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-h6xcf   1/1     Running   0          2m32s
nginx-deployment-6d8cd7bf4b-qr2ds   1/1     Running   0          30s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          2m32s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-h6xcf   1/1     Running   0          2m34s
nginx-deployment-6d8cd7bf4b-qr2ds   1/1     Running   0          32s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          2m34s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl delete pod nginx-deployment-6d8cd7bf4b-h6xcf -n dev
pod "nginx-deployment-6d8cd7bf4b-h6xcf" deleted from dev namespace

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-hpwn7   1/1     Running   0          3s
nginx-deployment-6d8cd7bf4b-qr2ds   1/1     Running   0          55s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          2m57s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl run nginx-dev --image=nginx:latest -n dev
pod/nginx-dev created

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-hpwn7   1/1     Running             0          34s
nginx-deployment-6d8cd7bf4b-qr2ds   1/1     Running             0          86s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running             0          3m28s
nginx-dev                           0/1     ContainerCreating   0          2s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl scale deployment nginx-deployment --replicas=5 -n dev
deployment.apps/nginx-deployment scaled

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-hpwn7   1/1     Running   0          65s
nginx-deployment-6d8cd7bf4b-kz8gn   1/1     Running   0          4s
nginx-deployment-6d8cd7bf4b-qr2ds   1/1     Running   0          117s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          3m59s
nginx-deployment-6d8cd7bf4b-xfdbz   1/1     Running   0          4s
nginx-dev                           1/1     Running   0          33s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl scale deployment nginx-deployment --replicas=2 -n dev
deployment.apps/nginx-deployment scaled

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-hpwn7   1/1     Running   0          78s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          4m12s
nginx-dev                           1/1     Running   0          46s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl apply -f nginx-deployment.yaml -n dev
deployment.apps/nginx-deployment configured

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-5xlws   1/1     Running   0          4s
nginx-deployment-6d8cd7bf4b-hpwn7   1/1     Running   0          117s
nginx-deployment-6d8cd7bf4b-hqqnn   1/1     Running   0          4s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          4m51s
nginx-dev                           1/1     Running   0          85s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-5xlws   1/1     Running   0          7s
nginx-deployment-6d8cd7bf4b-hpwn7   1/1     Running   0          2m
nginx-deployment-6d8cd7bf4b-hqqnn   1/1     Running   0          7s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running   0          4m54s
nginx-dev                           1/1     Running   0          88s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev
deployment.apps/nginx-deployment image updated

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-6d8cd7bf4b-5xlws   1/1     Running             0          35s
nginx-deployment-6d8cd7bf4b-hpwn7   1/1     Running             0          2m28s
nginx-deployment-6d8cd7bf4b-vf6cr   1/1     Running             0          5m22s
nginx-deployment-77bf8679f9-42bvf   0/1     ContainerCreating   0          5s
nginx-deployment-77bf8679f9-lnz7l   0/1     ContainerCreating   0          5s
nginx-dev                           1/1     Running             0          116s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl rollout status deployment/nginx-deployment -n dev
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -n dev
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77bf8679f9-42bvf   1/1     Running   0          39s
nginx-deployment-77bf8679f9-jnbzb   1/1     Running   0          5s
nginx-deployment-77bf8679f9-lnz7l   1/1     Running   0          39s
nginx-deployment-77bf8679f9-tphxw   1/1     Running   0          6s
nginx-dev                           1/1     Running   0          2m30s

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl rollout history deployment/nginx-deployment -n dev
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>


Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl rollout history deployment/nginx-deployment -n dev
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>


Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl rollout undo deployment/nginx-deployment -n dev
deployment.apps/nginx-deployment rolled back

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl rollout status deployment/nginx-deployment -n dev
deployment "nginx-deployment" successfully rolled out

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl rollout status deployment/nginx-deployment -n dev
deployment "nginx-deployment" successfully rolled out

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl describe deployment nginx-deployment -n dev | grep Image
    Image:         nginx:1.24

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl delete deployment nginx-deployment -n dev
deployment.apps "nginx-deployment" deleted from dev namespace

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl delete pod nginx-dev -n dev
kubectl delete pod nginx-staging -n staging
kubectl delete namespace dev staging production
pod "nginx-dev" deleted from dev namespace
Error from server (NotFound): pods "nginx-staging" not found
namespace "dev" deleted
namespace "staging" deleted
Error from server (NotFound): namespaces "production" not found

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   7h49m
kube-node-lease   Active   7h49m
kube-public       Active   7h49m
kube-system       Active   7h49m

Rahul@LAPTOP-KP49HN62 MINGW64 /d/kubernetes/pods
$ kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS        AGE
kube-system   coredns-66bc5c9577-c8lmr           1/1     Running   1 (7h48m ago)   7h49m
kube-system   etcd-minikube                      1/1     Running   1 (7h48m ago)   7h49m
kube-system   kube-apiserver-minikube            1/1     Running   1 (7h48m ago)   7h49m
kube-system   kube-controller-manager-minikube   1/1     Running   1 (7h49m ago)   7h49m
kube-system   kube-proxy-jc6qg                   1/1     Running   1 (7h49m ago)   7h49m
kube-system   kube-scheduler-minikube            1/1     Running   1 (7h49m ago)   7h49m
kube-system   storage-provisioner                1/1     Running   3 (17m ago)     7h49m
```