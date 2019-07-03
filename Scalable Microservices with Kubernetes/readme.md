## Scalable Microservices with Kubernetes

https://www.udacity.com/course/scalable-microservices-with-kubernetes--ud615

### Google Compute Engine

#### Shell

```bash
gcloud compute zones list
gcloud config set compute/zone `ZONE`

gcloud config list
```

### 12 Factor https://12factor.net/

<iframe src="//www.slideshare.net/slideshow/embed_code/key/tVU2OGq2bPzF1P" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/rudiyardley/the-12-factor-app" title="The 12 Factor App" target="_blank">The 12 Factor App</a> </strong> from <strong><a href="https://www.slideshare.net/rudiyardley" target="_blank">rudiyardley</a></strong> </div>
![Image result for twelve factor app](https://0901.static.prezi.com/preview/v2/7ojxhzhougi3uurlmhz57gja4d6jc3sachvcdoaizecfr3dnitcq_3_0.png)
https://prezi.com/8uldpq91vm4e/the-twelve-factor-app/

### JWT (Json Web Token)

Uses:

- Authentication
- Information Exchnage

https://jwt.io/

> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpXVCBtYWRlIGVhc3kiLCJhZG1pbiI6dHJ1ZX0.RhS5_R99IA0u_UffKr8xDh05Ob9Lb-kOBlmOWlspcc0

```json
PAYLOAD: {
  "sub": "1234567890",
  "name": "JWT made easy",
  "admin": true
}
```

### Kubernetes

```bash
#Launch a single instance:
kubectl run nginx --image=nginx:1.10.0

#Get pods
kubectl get pods

#Expose nginx
kubectl expose deployment nginx --port 80 --type LoadBalancer

#List services
kubectl get services
```

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

#### Pods

```bash
$cat pods/monolith.yaml
apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: udacity/example-monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"

$kubectl create -f pods/monolith.yaml
pod/monolith created

$kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
monolith                 1/1     Running   0          5s

#port-forwarding
$kubectl port-forward monolith 10080:80
Forwarding from 127.0.0.1:10080 -> 80

#run an interactive shell
$kubectl exec monolith --stdin --tty -c monolith /bin/sh
```

#### MHC (Monitoring and Health Checks)

```bash
$cat healthy-monolith.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "healthy-monolith"
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: udacity/example-monolith:1.0.0
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
      livenessProbe:
        httpGet:
          path: /healthz
          port: 81
          scheme: HTTP
        initialDelaySeconds: 5
        periodSeconds: 15
        timeoutSeconds: 5
      readinessProbe:
        httpGet:
          path: /readiness
          port: 81
          scheme: HTTP
        initialDelaySeconds: 5
        timeoutSeconds: 1
```

#### Secret

```bash
$kubectl create secret generic tls-certs --from-file=tls/
secret/tls-certs created

$kubectl describe secrets tls-certs
Name:         tls-certs
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
cert.pem:                 1253 bytes
key.pem:                  1679 bytes
ssl-extensions-x509.cnf:  275 bytes
update-tls.sh:            610 bytes
ca-key.pem:               1675 bytes
ca.pem:                   1099 bytes

$kubectl create configmap nginx-proxy-conf --from-file=nginx/proxy.conf
configmap/nginx-proxy-conf created

$kubectl describe configmap nginx-proxy-conf
Name:         nginx-proxy-conf
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
proxy.conf:
----
server {
  listen 443;
  ssl    on;

  ssl_certificate     /etc/tls/cert.pem;
  ssl_certificate_key /etc/tls/key.pem;

  location / {
    proxy_pass http://127.0.0.1:80;
  }
}

Events:  <none>
```

Config docs - http://kubernetes.io/docs/user-guide/configmap/
Secrets - http://kubernetes.io/docs/user-guide/secrets/

[![HTTP, HTTPS, SSL / TLS Explained](https://img.youtube.com/vi/hExRDVZHhig/0.jpg)](https://www.youtube.com/watch?v=hExRDVZHhig)

```bash
$cat pods/secure-monolith.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "secure-monolith"
  labels:
    app: monolith
spec:
  containers:
    - name: nginx
      image: "nginx:1.9.14"
      lifecycle:
        preStop:
          exec:
            command: ["/usr/sbin/nginx","-s","quit"]
      volumeMounts:
        - name: "nginx-proxy-conf"
          mountPath: "/etc/nginx/conf.d"
        - name: "tls-certs"
          mountPath: "/etc/tls"
    - name: monolith
      image: "udacity/example-monolith:1.0.0"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
      livenessProbe:
        httpGet:
          path: /healthz
          port: 81
          scheme: HTTP
        initialDelaySeconds: 5
        periodSeconds: 15
        timeoutSeconds: 5
      readinessProbe:
        httpGet:
          path: /readiness
          port: 81
          scheme: HTTP
        initialDelaySeconds: 5
        timeoutSeconds: 1
  volumes:
    - name: "tls-certs"
      secret:
        secretName: "tls-certs"
    - name: "nginx-proxy-conf"
      configMap:
        name: "nginx-proxy-conf"
        items:
          - key: "proxy.conf"
            path: "proxy.conf"

$curl https://127.0.0.1:10443
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.

$curl --cacert tls/ca.pem https://127.0.0.1:10443
{"message":"Hello"}

$kubectl logs -c nginx secure-monolith
127.0.0.1 - - [30/Jun/2019:20:53:24 +0000] "GET / HTTP/1.1" 200 20 "-" "curl/7.52.1" "-"
```

#### Deployments

```bash
$cat deployments/auth.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: auth
        track: stable
    spec:
      containers:
        - name: auth
          image: "udacity/example-auth:1.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: "10Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1

$kubectl create -f deployments/auth.yaml
deployment.extensions/auth created

$kubectl describe deployments auth
Name:                   auth
Namespace:              default
CreationTimestamp:      Sun, 30 Jun 2019 17:11:38 -0400
Labels:                 app=auth
                        track=stable
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=auth,track=stable
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  app=auth
           track=stable
  Containers:
   auth:
    Image:       udacity/example-auth:1.0.0
    Ports:       80/TCP, 81/TCP
    Host Ports:  0/TCP, 0/TCP
    Limits:
      cpu:        200m
      memory:     10Mi
    Liveness:     http-get http://:81/healthz delay=5s timeout=5s period=15s #success=1 #failure=3
    Readiness:    http-get http://:81/readiness delay=5s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   auth-7fbdb977c6 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  15s   deployment-controller  Scaled up replica set auth-7fbdb977c6 to 1

$kubectl create -f services/auth.yaml
service/auth created
```

```bash
$cat nginx/frontend.conf
upstream hello {
    server hello.default.svc.cluster.local;
}

upstream auth {
    server auth.default.svc.cluster.local;
}

server {
    listen 443;
    ssl    on;

    ssl_certificate     /etc/tls/cert.pem;
    ssl_certificate_key /etc/tls/key.pem;

    location / {
        proxy_pass http://hello;
    }

    location /login {
        proxy_pass http://auth;
    }
}

$kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
configmap/nginx-frontend-conf created

$kubectl create -f deployments/frontend.yaml
deployment.extensions/frontend created

$kubectl create -f services/frontend.yaml
service/frontend created

$kubectl get services frontend
NAME       TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)         AGE
frontend   LoadBalancer   10.83.3.233   35.203.68.161   443:31319/TCP   3m

$curl -k https://35.203.68.161
{"message":"Hello"}

$curl -k https://35.203.68.161/login -u user
Enter host password for user 'user':
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJleHAiOjE1NjIxODg4MTIsImlhdCI6MTU2MTkyOTYxMiwiaXNzIjoiYXV0aC5zZXJ2aWNlIiwic3ViIjoidXNlciJ9.aqaF3PBcIqchHo1x9lWAc5V8-7cNMbhds1l1hReFkgw"}
```

#### Scaling

```bash
$kubectl get replicasets
NAME                  DESIRED   CURRENT   READY   AGE
hello-794d79df4       1         1         1       14m

$kubectl get pods -l "app=hello,track=stable"
NAME                    READY   STATUS    RESTARTS   AGE
hello-794d79df4-trbcl   1/1     Running   0          9m51s

$vim deployments/hello.yaml
spec > replicas : 1 to 3

$kubectl apply -f deployments/hello.yaml
deployment.extensions/hello configured

$kubectl get replicasets
NAME                  DESIRED   CURRENT   READY   AGE
hello-794d79df4       3         3         2       14m

$kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
hello-794d79df4-6cj94       0/1     Pending   0          2m2s
hello-794d79df4-p2lxb       1/1     Running   0          2m2s
hello-794d79df4-trbcl       1/1     Running   0          15m

$kubectl describe deployments hello | grep Replicas:
Replicas:               3 desired | 3 updated | 3 total | 2 available | 1 unavailable
```

#### Rolling Updates

```bash
$vim deployments/auth.yaml
spec > templates > spec > containers > image > 1.0.0 to 2.0.0

$kubectl apply -f deployments/auth.yaml
deployment.extensions/auth configured

$kubectl describe deployments auth | grep -E "Strategy|NewReplicaSet"
StrategyType:           RollingUpdate
RollingUpdateStrategy:  1 max unavailable, 1 max surge
NewReplicaSet:   auth-5fd954d8f (1/1 replicas created)

$kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
auth-5fd954d8f-m44kx        0/1     Pending   0          3m44s

$kubectl describe pods auth-5fd954d8f-m44kx | grep Image:
    Image:       udacity/example-auth:2.0.0


```
