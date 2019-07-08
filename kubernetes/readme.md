## [Kubernetes Basic](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

### Create a Cluster

```json
$ kubectl version
Client Version:version.Info{  
   Major:"1",
   Minor:"15",
   GitVersion:"v1.15.0",
   GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529",
   GitTreeState:"clean",
   BuildDate:"2019-06-19T16:40:16Z",
   GoVersion:"go1.12.5",
   Compiler:"gc",
   Platform:"linux/amd64"
}
Server Version:version.Info{  
   Major:"1",
   Minor:"15",
   GitVersion:"v1.15.0",
   GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529",
   GitTreeState:"clean",
   BuildDate:"2019-06-19T16:32:14Z",
   GoVersion:"go1.12.5",
   Compiler:"gc",
   Platform:"linux/amd64"
}
```

```bash
$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   6m31s   v1.15.0
```

### Deploy app
```bash
$ kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --port=8080
```

```bash
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   0/1     0            0           6s

$ kubectl proxy
Starting to serve on 127.0.0.1:8001

$ curl http://localhost:8001/version
{
  "major": "1",
  "minor": "15",
  "gitVersion": "v1.15.0",
  "gitCommit": "e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529",
  "gitTreeState": "clean",
  "buildDate": "2019-06-19T16:32:14Z",
  "goVersion": "go1.12.5",
  "compiler": "gc",
  "platform": "linux/amd64"
}

$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
`kubernetes-bootcamp-5b48cfdcbd-vnppv`   1/1     Running   0          2m39s

$ curl http://localhost:8001/api/v1/namespaces/default/pods/kubernetes-bootcamp-5b48cfdcbd-vnppv/proxy/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5b48cfdcbd-vnppv | v=1
```

### Explorer app
```json
Cluster : {
	Master: {},
	Nodes: []
}
Node : {
	Pods: [{
        volume, // Shared storage
        ip address,
        containerized app
	}, ... {}}]
}
```

```bash
$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5b48cfdcbd-s5g2h   1/1     Running   0          72s

$ kubectl describe pods
Name:           kubernetes-bootcamp-5b48cfdcbd-s5g2h
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.35
Start Time:     Sat, 29 Jun 2019 20:26:36 +0000
Labels:         pod-template-hash=5b48cfdcbd
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b48cfdcbd
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://88a314769ccc66cf25556b63e067539d025f8f59a3c3c641e20e22759e2af4e5
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 20:26:37 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-58svs (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-58svs:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-58svs
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Warning  FailedScheduling  78s (x2 over 78s)  default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
  Normal   Scheduled         76s                default-scheduler  Successfully assigned default/kubernetes-bootcamp-5b48cfdcbd-s5g2h to minikube
  Normal   Pulled            75s                kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal   Created           75s                kubelet, minikube  Created container kubernetes-bootcamp
  Normal   Started           75s                kubelet, minikube  Started container kubernetes-bootcamp

$ kubectl logs kubernetes-bootcamp-5b48cfdcbd-s5g2h
Kubernetes Bootcamp App Started At: 2019-06-29T20:26:37.556Z | Running On:  kubernetes-bootcamp-5b48cfdcbd-s5g2h

Running On: kubernetes-bootcamp-5b48cfdcbd-s5g2h | Total Requests: 1 | App Uptime: 268.385 seconds | Log Time: 2019-06-29T20:31:05.941Z

$ kubectl exec -ti kubernetes-bootcamp-5b48cfdcbd-s5g2h bash

$ cat server.js
var http = require('http');
var requests=0;
var podname= process.env.HOSTNAME;
var startTime;
var host;
var handleRequest = function(request, response) {
  response.setHeader('Content-Type', 'text/plain');
  response.writeHead(200);
  response.write("Hello Kubernetes bootcamp! | Running on: ");
  response.write(host);
  response.end(" | v=1\n");
  console.log("Running On:" ,host, "| Total Requests:", ++requests,"| App Uptime:", (new Date() - startTime)/1000 , "seconds", "| Log Time:",new Date());
}
var www = http.createServer(handleRequest);
www.listen(8080,function () {
    startTime = new Date();;
    host = process.env.HOSTNAME;
    console.log ("Kubernetes Bootcamp App Started At:",startTime, "| Running On: " ,host, "\n" );
});
```

### Expose App
```json
Service : ClusterIP | NodePort | LoadBalancer | ExternalName
```
[Connecting Applications with Services](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service)

```bash
$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
service/kubernetes-bootcamp exposed

$ kubectl get services
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1      <none>        443/TCP          114s
kubernetes-bootcamp   NodePort    10.97.154.22   <none>        8080:32418/TCP   107s

$ kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.97.154.22
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  `32418`/TCP
Endpoints:                172.18.0.2:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
$ echo NODE_PORT=$NODE_PORT
NODE_PORT=32418

$ curl $(minikube ip):32418
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5b48cfdcbd-cbvg7 | v=1

## Using Labels

$ kubectl describe deployment
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Sat, 29 Jun 2019 20:51:55 +0000
Labels:                 run=kubernetes-bootcamp
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=kubernetes-bootcamp
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=kubernetes-bootcamp
  Containers:
   kubernetes-bootcamp:
    Image:        gcr.io/google-samples/kubernetes-bootcamp:v1
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   kubernetes-bootcamp-5b48cfdcbd (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  8m53s  deployment-controller  Scaled up replica set kubernetes-bootcamp-5b48cfdcbd to 1
  
  
$ kubectl get pods -l run=kubernetes-bootcamp
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5b48cfdcbd-cbvg7   1/1     Running   0          10m

$ kubectl get services -l run=kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.97.154.22   <none>        8080:32418/TCP   11m

$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')

$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5b48cfdcbd-cbvg7

$ kubectl label pod $POD_NAME app=v1
pod/kubernetes-bootcamp-5b48cfdcbd-cbvg7 labeled

$ kubectl describe pods $POD_NAME
Name:           kubernetes-bootcamp-5b48cfdcbd-cbvg7
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.8
Start Time:     Sat, 29 Jun 2019 20:52:03 +0000
Labels:         `app=v1`
                pod-template-hash=5b48cfdcbd
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b48cfdcbd
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://6864684d14c5e367b5fe1b9226721ea7ea3a43ee4a1eea54eb2b6cd7ac260441
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 20:52:05 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:         /var/run/secrets/kubernetes.io/serviceaccount from default-token-kbxfm (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-kbxfm:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-kbxfm
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Warning  FailedScheduling  13m (x2 over 13m)  default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
  Normal   Scheduled         13m                default-scheduler  Successfully assigned default/kubernetes-bootcamp-5b48cfdcbd-cbvg7 to minikube
  Normal   Pulled            13m                kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal   Created           13m                kubelet, minikube  Created container kubernetes-bootcamp
  Normal   Started           13m                kubelet, minikube  Started container kubernetes-bootcamp

$ kubectl get pods -l `app=v1`
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5b48cfdcbd-cbvg7   1/1     Running   0          17m

## Delete Service

$ kubectl delete service -l run=kubernetes-bootcamp
service "kubernetes-bootcamp" deleted

$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   19m

$ curl $(minikube ip):$NODE_PORT  ## not reachable anymore from outside of the cluster
curl: (7) Failed to connect to 172.17.0.8 port 32418: Connection refused

$ kubectl exec -ti $POD_NAME curl localhost:8080  ## but it's still running with a curl inside the pod
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5b48cfdcbd-cbvg7 | v=1
```

### Scale App

#### Scale Up
```bash
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   0/1     0            0           4s

$ kubectl scale deployments/kubernetes-bootcamp --replicas=4
deployment.extensions/kubernetes-bootcamp scaled

$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4/4     4            4           32s

$ kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
kubernetes-bootcamp-5b48cfdcbd-9sjzc   1/1     Running   0          22s   172.18.0.6   minikube   <none>           <none>
kubernetes-bootcamp-5b48cfdcbd-hvmnm   1/1     Running   0          41s   172.18.0.4   minikube   <none>           <none>
kubernetes-bootcamp-5b48cfdcbd-t8gvv   1/1     Running   0          22s   172.18.0.5   minikube   <none>           <none>
kubernetes-bootcamp-5b48cfdcbd-zxhxq   1/1     Running   0          22s   172.18.0.7   minikube   <none>           <none>

$ kubectl describe deployments/kubernetes-bootcamp
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Sat, 29 Jun 2019 21:31:35 +0000
Labels:                 run=kubernetes-bootcamp
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=kubernetes-bootcamp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=kubernetes-bootcamp
  Containers:
   kubernetes-bootcamp:
    Image:        gcr.io/google-samples/kubernetes-bootcamp:v1
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   kubernetes-bootcamp-5b48cfdcbd (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  58s   deployment-controller  Scaled up replica set kubernetes-bootcamp-5b48cfdcbd to 1
  Normal  ScalingReplicaSet  39s   deployment-controller  Scaled up replica set kubernetes-bootcamp-5b48cfdcbd to 4
```
#### Load Balancing

```bash
$ kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.101.124.56
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  31460/TCP
Endpoints:                172.18.0.4:8080,172.18.0.5:8080,172.18.0.6:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')

$ echo NODE_PORT=$NODE_PORT
NODE_PORT=31460

$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5b48cfdcbd-hvmnm | v=1
```

#### Scale Down
```bash
$ kubectl scale deployments/kubernetes-bootcamp --replicas=2
deployment.extensions/kubernetes-bootcamp scaled

$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   2/2     2            2           9m12s

$ kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
kubernetes-bootcamp-5b48cfdcbd-hvmnm   1/1     Running   0          9m7s    172.18.0.4   minikube   <none>           <none>
kubernetes-bootcamp-5b48cfdcbd-t8gvv   1/1     Running   0          8m48s   172.18.0.5   minikube   <none>           <none>
```

### Rolling Update

![](https://www.bluematador.com/hs-fs/hubfs/blog/new/Kubernetes%20Deployments%20-%20Rolling%20Update%20Configuration/Kubernetes-Deployments-Rolling-Update-Configuration.gif?width=1600&name=Kubernetes-Deployments-Rolling-Update-Configuration.gif) https://www.bluematador.com/blog/kubernetes-deployments-rolling-update-configuration

```bash
$ kubectl get depkubectl get deployments
error: the server doesn't have a resource type "depkubectl"

$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4/4     4            4           14s

$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5b48cfdcbd-6wdbn   1/1     Running   0          16s
kubernetes-bootcamp-5b48cfdcbd-8jvgq   1/1     Running   0          16s
kubernetes-bootcamp-5b48cfdcbd-ckkpd   1/1     Running   0          16s
kubernetes-bootcamp-5b48cfdcbd-kvm8n   1/1     Running   0          16s

$ kubectl describe pods
Name:           kubernetes-bootcamp-5b48cfdcbd-6wdbn
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:53:37 +0000
Labels:         pod-template-hash=5b48cfdcbd
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.7
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b48cfdcbd
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://812c50b0d01cc7d47d5c59b1815fba51339fc1d0ab736087ee8fcbd25a9002e3
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:53:39 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  20s   default-scheduler  Successfully assigned default/kubernetes-bootcamp-5b48cfdcbd-6wdbn to minikube
  Normal  Pulled     19s   kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal  Created    19s   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    18s   kubelet, minikube  Started container kubernetes-bootcamp

Name:           kubernetes-bootcamp-5b48cfdcbd-8jvgq
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:53:37 +0000
Labels:         pod-template-hash=5b48cfdcbd
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b48cfdcbd
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://89f62abe985127e4c0898515b176450bb2839b33894a31b3ddec142024b886e7
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:53:39 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  20s   default-scheduler  Successfully assigned default/kubernetes-bootcamp-5b48cfdcbd-8jvgq to minikube
  Normal  Pulled     19s   kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal  Created    19s   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    18s   kubelet, minikube  Started container kubernetes-bootcamp

Name:           kubernetes-bootcamp-5b48cfdcbd-ckkpd
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:53:37 +0000
Labels:         pod-template-hash=5b48cfdcbd
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.3
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b48cfdcbd
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://62aca7acdade5b35d9f05ba944c72af435e725f6c8d9fd689eb3dd286b7b463c
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:53:39 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  20s   default-scheduler  Successfully assigned default/kubernetes-bootcamp-5b48cfdcbd-ckkpd to minikube
  Normal  Pulled     19s   kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal  Created    19s   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    18s   kubelet, minikube  Started container kubernetes-bootcamp

Name:           kubernetes-bootcamp-5b48cfdcbd-kvm8n
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:53:37 +0000
Labels:         pod-template-hash=5b48cfdcbd
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.6
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b48cfdcbd
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://e4506dfb1614ff2b07d602b6ca898fca13f910effe3fd5899aa6a2296447624c
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:53:39 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  20s   default-scheduler  Successfully assigned default/kubernetes-bootcamp-5b48cfdcbd-kvm8n to minikube
  Normal  Pulled     19s   kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal  Created    19s   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    18s   kubelet, minikube  Started container kubernetes-bootcamp

$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
deployment.extensions/kubernetes-bootcamp image updated

$ kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-5b48cfdcbd-6wdbn   1/1     Terminating   0          44s
kubernetes-bootcamp-5b48cfdcbd-8jvgq   1/1     Terminating   0          44s
kubernetes-bootcamp-5b48cfdcbd-ckkpd   1/1     Terminating   0          44s
kubernetes-bootcamp-5b48cfdcbd-kvm8n   1/1     Terminating   0          44s
kubernetes-bootcamp-cfc74666-cdcck     1/1     Running       0          13s
kubernetes-bootcamp-cfc74666-hmk5j     1/1     Running       0          15s
kubernetes-bootcamp-cfc74666-lpxgl     1/1     Running       0          15s
kubernetes-bootcamp-cfc74666-xjmrs     1/1     Running       0          13s

$ kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.104.130.182
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30075/TCP
Endpoints:                172.18.0.10:8080,172.18.0.11:8080,172.18.0.8:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')

$ echo NODE_PORT=$NODE_PORT
NODE_PORT=30075

$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-cfc74666-lpxgl | v=2

$ kubectl rollout status deployments/kubernetes-bootcamp
deployment "kubernetes-bootcamp" successfully rolled out

$ kubectl describe pods
Name:           kubernetes-bootcamp-cfc74666-cdcck
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:54:08 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.10
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://b4c372113028041c3a4ba75796fd7dc7298aab78f14c300255b0f25ea9ca5d3a
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:09 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:         /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m19s  default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-cdcck to minikube
  Normal  Pulled     3m18s  kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    3m18s  kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    3m18s  kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-cfc74666-hmk5j
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:54:06 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.9
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://db6c13ee7a5b1f11ed8b38b4a0c8f526a56166733860af19b83cf3e3c5e51b7d
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:07 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:         /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m21s  default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-hmk5j to minikube
  Normal  Pulled     3m20s  kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    3m20s  kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    3m20s  kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-cfc74666-lpxgl
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:54:06 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.8
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://ceed8871dc98038e1fc5da8d08aa8405a35f1fd0efaa3180fd3f07c92cb8890f
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:07 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:         /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m21s  default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-lpxgl to minikube
  Normal  Pulled     3m20s  kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    3m20s  kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    3m20s  kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-cfc74666-xjmrs
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:54:08 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.11
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://9f4732fe8505d082325b0c114d34d65490b262cb1a74a01303a587d364d44e18
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:         /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m19s  default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-xjmrs to minikube
  Normal  Pulled     3m18s  kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    3m18s  kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    3m17s  kubelet, minikube  Started container kubernetes-bootcamp
```

#### Rollback

```bash
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
deployment.extensions/kubernetes-bootcamp image updated

$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   3/4     2            3           11m

$ kubectl get pods
NAME                                   READY   STATUS             RESTARTS   AGE
kubernetes-bootcamp-547469f5dd-j7ns8   0/1     ImagePullBackOff   0          18s
kubernetes-bootcamp-547469f5dd-vshrd   0/1     ImagePullBackOff   0          18s
kubernetes-bootcamp-cfc74666-cdcck     1/1     Running            0          10m
kubernetes-bootcamp-cfc74666-hmk5j     1/1     Running            0          10m
kubernetes-bootcamp-cfc74666-lpxgl     1/1     Running            0          10m
kubernetes-bootcamp-cfc74666-xjmrs     1/1     Terminating        0          10m

$ kubectl describe pods
Name:           kubernetes-bootcamp-547469f5dd-j7ns8
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 22:04:20 +0000
Labels:         pod-template-hash=547469f5dd
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Pending
IP:             172.18.0.3
Controlled By:  ReplicaSet/kubernetes-bootcamp-547469f5dd
Containers:
  kubernetes-bootcamp:
    Container ID:
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v10
    Image ID:
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:         /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  25s               default-scheduler  Successfully assigned default/kubernetes-bootcamp-547469f5dd-j7ns8 to minikube
  Normal   BackOff    21s               kubelet, minikube  Back-off pulling image "gcr.io/google-samples/kubernetes-bootcamp:v10"
  Warning  Failed     21s               kubelet, minikube  Error: ImagePullBackOff
  Normal   Pulling    9s (x2 over 24s)  kubelet, minikube  Pulling image "gcr.io/google-samples/kubernetes-bootcamp:v10"
  Warning  Failed     9s (x2 over 23s)  kubelet, minikube  Failed to pull image "gcr.io/google-samples/kubernetes-bootcamp:v10": rpc error: code = Unknown desc = Error response from daemon: manifest for gcr.io/google-samples/kubernetes-bootcamp:v10 not found
  Warning  Failed     9s (x2 over 23s)  kubelet, minikube  Error: ErrImagePull


Name:           kubernetes-bootcamp-547469f5dd-vshrd
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 22:04:20 +0000
Labels:         pod-template-hash=547469f5dd
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Pending
IP:             172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-547469f5dd
Containers:
  kubernetes-bootcamp:
    Container ID:
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v10
    Image ID:
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:         /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  25s                default-scheduler  Successfully assigned default/kubernetes-bootcamp-547469f5dd-vshrd to minikube
  Normal   BackOff    21s (x2 over 23s)  kubelet, minikube  Back-off pulling image "gcr.io/google-samples/kubernetes-bootcamp:v10"
  Warning  Failed     21s (x2 over 23s)  kubelet, minikube  Error: ImagePullBackOff
  Normal   Pulling    8s (x2 over 24s)   kubelet, minikube  Pulling image "gcr.io/google-samples/kubernetes-bootcamp:v10"
  Warning  Failed     8s (x2 over 23s)   kubelet, minikube  Failed to pull image "gcr.io/google-samples/kubernetes-bootcamp:v10": rpc error: code = Unknown desc = Error response from daemon: manifest for gcr.io/google-samples/kubernetes-bootcamp:v10 not found
  Warning  Failed     8s (x2 over 23s)   kubelet, minikube  Error: ErrImagePull


Name:           kubernetes-bootcamp-cfc74666-cdcck
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:54:08 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.10
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://b4c372113028041c3a4ba75796fd7dc7298aab78f14c300255b0f25ea9ca5d3a
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:09 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:         	/var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-cdcck to minikube
  Normal  Pulled     10m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    10m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    10m   kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-cfc74666-hmk5j
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:54:06 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.9
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://db6c13ee7a5b1f11ed8b38b4a0c8f526a56166733860af19b83cf3e3c5e51b7d
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:07 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-hmk5j to minikube
  Normal  Pulled     10m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    10m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    10m   kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-cfc74666-lpxgl
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:54:06 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.8
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://ceed8871dc98038e1fc5da8d08aa8405a35f1fd0efaa3180fd3f07c92cb8890f
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:07 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-lpxgl to minikube
  Normal  Pulled     10m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    10m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    10m   kubelet, minikube  Started container kubernetes-bootcamp


Name:                      kubernetes-bootcamp-cfc74666-xjmrs
Namespace:                 default
Priority:                  0
Node:                      minikube/172.17.0.14
Start Time:                Sat, 29 Jun 2019 21:54:08 +0000
Labels:                    pod-template-hash=cfc74666
                           run=kubernetes-bootcamp
Annotations:               <none>
Status:                    Terminating (lasts <invalid>)
Termination Grace Period:  30s
IP:                        172.18.0.11
Controlled By:             ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://9f4732fe8505d082325b0c114d34d65490b262cb1a74a01303a587d364d44e18
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-xjmrs to minikube
  Normal  Pulled     10m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    10m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    10m   kubelet, minikube  Started container kubernetes-bootcamp
  Normal  Killing    25s   kubelet, minikube  Stopping container kubernetes-bootcamp

$ kubectl rollout undo deployments/kubernetes-bootcamp
deployment.extensions/kubernetes-bootcamp rolled back

$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-cfc74666-cdcck   1/1     Running   0          11m
kubernetes-bootcamp-cfc74666-hmk5j   1/1     Running   0          11m
kubernetes-bootcamp-cfc74666-lpxgl   1/1     Running   0          11m
kubernetes-bootcamp-cfc74666-t5wns   1/1     Running   0          5s

$ kubectl describe pods
Name:           kubernetes-bootcamp-cfc74666-cdcck
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:54:08 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.10
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://b4c372113028041c3a4ba75796fd7dc7298aab78f14c300255b0f25ea9ca5d3a
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:09 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  11m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-cdcck to minikube
  Normal  Pulled     11m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    11m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    11m   kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-cfc74666-hmk5j
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:54:06 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.9
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://db6c13ee7a5b1f11ed8b38b4a0c8f526a56166733860af19b83cf3e3c5e51b7d
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:07 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  11m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-hmk5j to minikube
  Normal  Pulled     11m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    11m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    11m   kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-cfc74666-lpxgl
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 21:54:06 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.8
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://ceed8871dc98038e1fc5da8d08aa8405a35f1fd0efaa3180fd3f07c92cb8890f
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 21:54:07 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  11m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-lpxgl to minikube
  Normal  Pulled     11m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    11m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    11m   kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-cfc74666-t5wns
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.14
Start Time:     Sat, 29 Jun 2019 22:05:05 +0000
Labels:         pod-template-hash=cfc74666
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.6
Controlled By:  ReplicaSet/kubernetes-bootcamp-cfc74666
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://3b4d27752a1a09d879b2a33eadd7c7a7c2a52f0d8f4542ca6de7abc94f924f59
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Jun 2019 22:05:06 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fzgrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fzgrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fzgrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  8s    default-scheduler  Successfully assigned default/kubernetes-bootcamp-cfc74666-t5wns to minikube
  Normal  Pulled     8s    kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created    7s    kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    7s    kubelet, minikube  Started container kubernetes-bootcamp
  
```





