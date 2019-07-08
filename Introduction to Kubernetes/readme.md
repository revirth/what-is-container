# Introduction to Kubernetes



## Container Orchestration
[![](https://cloudify.co/wp-content/uploads/2018/01/cloudify-orch-new-1024x372.png)](https://www.google.ca/url?sa=i&source=images&cd=&ved=2ahUKEwiSuv6Ci5zjAhXQnOAKHcvSDE8QjRx6BAgBEAU&url=https%3A%2F%2Fwww.automationexchange.com%2Fproddir%2Fdir_product.asp%3Fdir_id%3D10%26p_id%3D2814&psig=AOvVaw1LfuXO2ElDv_sEo-YLyhdF&ust=1562357924167897)

## Container Orchestrators
1.  [Amazon Elastic Container Service](https://aws.amazon.com/ecs/) (ECS) runs Docker containers at scale on its infrastructure.
2.  [Azure Container Instance](https://azure.microsoft.com/en-us/services/container-instances/) (ACI) is a basic container orchestration service provided by [Microsoft Azure](https://azure.microsoft.com/en-us/).
3.  [Azure Service Fabric](https://azure.microsoft.com/en-us/services/service-fabric/) is an open source container orchestrator provided by [Microsoft Azure](https://azure.microsoft.com/en-us/).
4.  [Kubernetes](https://kubernetes.io/) is an open source orchestration tool, started by Google, part of the [Cloud Native Computing Foundation](https://www.cncf.io/) (CNCF) project.
5.  [Marathon](https://mesosphere.github.io/marathon/) is a framework to run containers at scale on [Apache Mesos](https://mesos.apache.org/).
6.  [Nomad](https://www.nomadproject.io/) is the container orchestrator provided by [HashiCorp](https://www.hashicorp.com/).
7.  [Docker Swarm](https://docs.docker.com/engine/swarm/) is a container orchestrator provided by [Docker, Inc](https://www.docker.com/). It is part of [Docker Engine](https://docs.docker.com/engine/).

## Kubernetes as-a-Service

- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) (GKE)
- [Amazon Elastic Container Service for Kubernetes](https://aws.amazon.com/eks/) (Amazon EKS)
- [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/) (AKS)
- [IBM Cloud Kubernetes Service](https://www.ibm.com/cloud/container-service)
- [DigitalOcean Kubernetes](https://www.digitalocean.com/products/kubernetes/)
- [Oracle Container Engine for Kubernetes](https://cloud.oracle.com/containers/kubernetes-engine)

##   Features

- **Automatic bin packing**
  Kubernetes automatically schedules containers based on resource needs and constraints, to maximize utilization without sacrificing availability.
- **Self-healing**
  Kubernetes automatically replaces and reschedules containers from failed nodes. It kills and restarts containers unresponsive to health checks, based on existing rules/policy. It also prevents traffic from being routed to unresponsive containers.
- **Horizontal scaling**
  With Kubernetes applications are scaled manually or automatically based on CPU or custom metrics utilization.
- **Service discovery and Load balancing**
  Containers receive their own IP addresses from Kubernetes, white it assigns a single Domain Name System (DNS) name to a set of containers to aid in load-balancing requests across the containers of the set.
- **Automated rollouts and rollbacks**
  Kubernetes seamlessly rolls out and rolls back application updates and configuration changes, constantly monitoring the application's health to prevent any downtime.
- **Secret and configuration management**
  Kubernetes manages secrets and configuration details for an application separately from the container image, in order to avoid a re-build of the respective image. Secrets consist of confidential information passed to the application without revealing the sensitive content to the stack configuration, like on GitHub.
- **Storage orchestration**
  Kubernetes automatically mounts software-defined storage (SDS) solutions to containers from local storage, external cloud providers, or network storage systems.
- **Batch execution**
  Kubernetes supports batch execution, long-running jobs, and replaces failed containers.

##  Master Node
![](https://pbs.twimg.com/media/DQenEl3UEAAmNc9.jpg)
https://twitter.com/jpetazzo/status/938902771302060034

- **API Server** 

  - intercepts RESTful calls from users, operators and external agents, then validates and processes
  - reads cluster's current state from the **etcd**, and after a call's execution, the resulting state of the Kubernetes cluster is saved in the distributed key-value data store for persistence
  - acts as a middle-man interface for any other control plane agent requiring to access the cluster's data store

- **Scheduler**

  - obtains from etcd, via the API server, resource usage data for each worker node in the cluster
  - receives from the API server the new object's requirements which are part of its configuration data

- **Controller Manager**

  - controls plane components on the master node running controllers to regulate the state of the Kubernetes cluster
  - are watch-loops continuously running and comparing the cluster's desired state (provided by objects' configuration data) with its current state (obtained from etcd data store via the API server)

- **etcd**

  - only the API server is able to communicate with the etcd data store

## Worker Node

![s](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/cca459b616974dca1face4a7a14808e7/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/Worker_Node.png)

- Container runtime

  - In order to run and manage a container's lifecycle, Kubernetes requires a **container runtime** on the node where a Pod and its containers are to be scheduled

- kubelet

  - communicates with the control plane components from the master node

  - receives Pod definitions, primarily from the API server, and interacts with the container runtime on the node to run containers associated with the Pod

  - monitors the health of the Pod's running containers

  - connects to the container runtime using **[Container Runtime Interface](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) (CRI)**

    - **ImageService** is responsible for all the image-related operations

    - **RuntimeService** is responsible for all the Pod and container-related operations

    - with the development of CRI, Kubernetes is more flexible now and uses different container runtimes without the need to recompile

    - ![](https://storage.googleapis.com/static.ianlewis.org/prod/img/772/CRI.png)https://www.ianlewis.org/en/container-runtimes-part-4-kubernetes-container-run

      

- kube-proxy

  - network agent which runs on each node responsible for dynamic updates and maintenance of all networking rules on the node

- Addons for DNS, Dashboard, cluster-level monitoring and logging

## Dashboard

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/kubernetes-metrics-scraper created

$ kubectl proxy
Starting to serve on 127.0.0.1:8001

$ TOKEN=$(kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t' | tr -d " ")

$ echo $TOKEN
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ...
```

## Building Blocks

- Pods

  - the unit of deployment in Kubernetes

  - represents a single instance of the application

  - logical collection of one or more containers

  - do not have the capability to self-heal by themselves

  - Pod object's configuration

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.11
        ports:
        - containerPort: 80
    ```

- Labels

  ![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/6669997d43534cbd2f251a57ebe0587c/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/Labels.png)

- Label Selectors

  ![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/2f83a85bea7ffd9fd861195725d7f146/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/Selectors.png)

- ReplicaSet
  - after detecting that the current state is no longer matching the desired state, ReplicaSet will create an additional Pod, thus ensuring that the current state matches the desired state

  ![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/010297d9a0a76859de8110273eb56ae1/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/ReplicaSet2.png)
  
  ![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/454b618e38084900bf247aecec0b3679/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/ReplicaSet3.png)

- Deployment - `rolling update`
  ![](https://www.bluematador.com/hs-fs/hubfs/blog/new/Kubernetes%20Deployments%20-%20Rolling%20Update%20Configuration/Kubernetes-Deployments-Rolling-Update-Configuration.gif?width=1600&name=Kubernetes-Deployments-Rolling-Update-Configuration.gif)https://www.bluematador.com/blog/kubernetes-deployments-rolling-update-configuration
- Namespaces
  - **default** Namespace contains the objects and resources created by administrators and developers  
  - **kube-public** used for special purposes such as exposing public (non-sensitive) information about the cluster
  - **kube-system** contains the objects created by the Kubernetes system, mostly the control plane agents
  - **kube-node-lease** holds node lease objects used for node heartbeat data

## Services
  ```YAML
kind: Service
apiVersion: v1
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
  ```
![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/1fba1b8cafc11dfea6e4c9069431c2dd/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/SOE.png)

 - `frontend-svc` Service has 3 endpoints: 10.0.1.3:5000, 10.0.1.4:5000, and 10.0.1.5:5000, created and managed automatically by the Service, not by the Kubernetes cluster administrator

### kube-proxy
 - watches the API server on the master node for the addition and removal of Services and endpoints
 ![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/f6184f33a4c81a2c59eb9c28bf79c3ae/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/kubeproxy.png)

### Service-Type
 - ClusterIP and NodePort
 - LoadBalancer - only work if the underlying infrastructure supports the automatic creation of Load Balancers and have the respective support in Kubernetes
 - ExternalIP - not managed by Kubernetes
 - ExternalName 

## Deploy an application using dashboard

| AppName         | webserver    |
|-----------------|--------------|
| Container Image | nginx:alpine |
| Number of pods  | 3            |
| Service         | none         |

- default, **k8s**-**app:webserver** label

```bash
$ kubectl get deployments
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
webserver    3         3         3            3           12m

$ kubectl get replicasets
NAME                    DESIRED   CURRENT   READY   AGE
webserver-74b684cccd    3         3         3       13m

$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
webserver-74b684cccd-b5kh9    1/1     Running   0          13m
webserver-74b684cccd-ktgmw    1/1     Running   0          13m
webserver-74b684cccd-rkzfl    1/1     Running   0          13m

$ kubectl get pods -L k8s-app,label2
NAME                          READY   STATUS    RESTARTS   AGE   K8S-APP     LABEL2
webserver-74b684cccd-b5kh9    1/1     Running   0          14m   webserver
webserver-74b684cccd-ktgmw    1/1     Running   0          14m   webserver
webserver-74b684cccd-rkzfl    1/1     Running   0          14m   webserver

$ kubectl get pods -l k8s-app=webserver
NAME                         READY   STATUS    RESTARTS   AGE
webserver-74b684cccd-b5kh9   1/1     Running   0          15m
webserver-74b684cccd-ktgmw   1/1     Running   0          15m
webserver-74b684cccd-rkzfl   1/1     Running   0          15m

$ kubectl delete deployments webserver
deployment.extensions "webserver" deleted
```

## Deploy an application using CLI

- webserver.yaml

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: webserver
    labels:
      app: nginx
  spec:
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
          image: nginx:alpine
          ports:
          - containerPort: 80
  ```

  ```bash
  $ kubectl create -f webserver.yaml
  deployment.apps/webserver created
  
  $ kubectl get replicasets
  NAME                    DESIRED   CURRENT   READY   AGE
  webserver-77d8994b6f    3         3         3       43s
  
  $ kubectl get pods
  NAME                          READY   STATUS    RESTARTS   AGE
  webserver-77d8994b6f-8qfq4    1/1     Running   0          52s
  webserver-77d8994b6f-9ghrh    1/1     Running   0          52s
  webserver-77d8994b6f-smgp7    1/1     Running   0          52s
  ```

- webserver-svc.yaml

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: web-service
      labels:
        run: web-service
    spec:
      type: NodePort
      ports:
      - port: 80
        protocol: TCP
      selector:
        app: nginx 
    ```

    ```bash
    $ kubectl create -f webserver-svc.yaml
    service/web-service created
    
    $ kubectl expose deployment webserver --name=web-service --type=NodePort
    Error from server (AlreadyExists): services "web-service" already exists
    
    $ kubectl get services
    NAME          TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    kubernetes    ClusterIP      10.96.0.1        <none>        443/TCP          5h
    web-service   NodePort       10.98.61.76      <none>        80:31246/TCP     4m
    
    $ kubectl describe service web-service
    Name:                     web-service
    Namespace:                default
    Labels:                   run=web-service
    Annotations:              <none>
    Selector:                 app=nginx
    Type:                     NodePort
    IP:                       10.98.61.76
    LoadBalancer Ingress:     localhost
    Port:                     <unset>  80/TCP
    TargetPort:               80/TCP
    NodePort:                 <unset>  31246/TCP
    Endpoints:                10.1.0.15:80,10.1.0.16:80,10.1.0.17:80
    Session Affinity:         None
    External Traffic Policy:  Cluster
    Events:                   <none>
    ```
    
## Volumes

- [Types of Volumes](https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes)

- Persistent Volume
  ![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/fcf7dcbb00433275fbbaa7bd5a8d400e/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/pvc2.png)

## ConfigMaps

- allow us to decouple the configuration details from the container image

```bash
$ kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
configmap/my-config created

$ kubectl get configmaps my-config
NAME        DATA   AGE
my-config   2      24s

$ kubectl get configmaps my-config -o yaml
apiVersion: v1
data:
  key1: value1
  key2: value2
kind: ConfigMap
metadata:
  creationTimestamp: "2019-07-06T23:18:46Z"
  name: my-config
  namespace: default
  resourceVersion: "27352"
  selfLink: /api/v1/namespaces/default/configmaps/my-config
  uid: 673fcb24-a044-11e9-b486-00155d547901
```

- customer1-configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: customer1
data:
  TEXT1: Customer1_Company
  TEXT2: Welcomes You
  COMPANY: Customer1 Company Technology Pct. Ltd.
```
```bash
$ kubectl create -f customer1-configmap.yaml
configmap/customer1 created

$ kubectl get configmap customer1
NAME        DATA   AGE
customer1   3      55s
```

- permission-reset.properties
```yaml
permission=read-only
allowed="true"
resetCount=3
```
```bash
$ kubectl create configmap permission-config --from-file=./permission-reset.properties
configmap/permission-config created
```

## Secrets

- allow us to encode the sensitive information like passwords, tokens, or keys before sharing it
- Secret data is stored as plain text inside **etcd**

```bash
$ kubectl create secret generic my-password --from-literal=password=mysqlpassword
secret/my-password created

$ kubectl get secret my-password
NAME          TYPE     DATA   AGE
my-password   Opaque   1      30s

$ kubectl describe secret my-password
Name:         my-password
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  13 bytes
```

- mypass.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-password
type: Opaque
stringData:
  password: mysqlpassword
```

```bash
$ kubectl get secrets my-password -o yaml
apiVersion: v1
data:
  password: bXlzcWxwYXNzd29yZA==
kind: Secret
metadata:
  creationTimestamp: "2019-07-06T23:38:14Z"
  name: my-password
  namespace: default
  resourceVersion: "28635"
  selfLink: /api/v1/namespaces/default/secrets/my-password
  uid: 1fabec26-a047-11e9-b486-00155d547901
type: Opaque
```

- password.txt
```bash
$ echo mysqlpassword | base64
bXlzcWxwYXNzd29yZAo=

$ echo -n 'bXlzcWxwYXNzd29yZAo=' > password.txt

$ kubectl create secret generic my-file-password --from-file=password.txt
secret/my-file-password created

$ kubectl get secrets my-file-password
NAME               TYPE     DATA   AGE
my-file-password   Opaque   1      1m

$ kubectl describe secrets my-file-password
Name:         my-file-password
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password.txt:  20 bytes
```

## Ingress

<a href="https://youtu.be/VicH6KojwCI?t=51" target="_blank"><img src="https://img.youtube.com/vi/VicH6KojwCI/0.jpg" alt="GraphQL: The Documentary (Official Release)"></a>

## Advanced Topic

- **Anotations** are not used to identify and select objects
- **jobs and cronjobs**
- **autoscaling**
- **daemonset** collects monitoring data from all nodes, or to run a storage daemon on all nodes
- **StatefulSet** controller provides identity and guaranteed ordering of deployment and scaling to Pods
- **Helm** is a package manager (analogous to **yum** and **apt** for Linux) for Kubernetes. we can bundle all manifests such as Deployments, Services, Volume Claims, Ingress, etc after templatizing them into a well-defined format, along with other metadata
- Monitoring and Logging
  - Kubernetes does not provide cluster-wide logging by default, therefore third party tools are required to centralize and aggregate cluster logs
  - The most common way to collect the logs is using [Elasticsearch](https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/), which uses [fluentd](http://www.fluentd.org/) with custom configuration as an agent on the nodes.

