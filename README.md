![logo](https://user-images.githubusercontent.com/30426958/61536054-69206080-aa3c-11e9-9675-8ec43432017b.png)
# Install Minikube
This page shows you how to install Minikube, a tool that runs a single-node Kubernetes cluster in a virtual machine on your personal computer.

#### Prerequisites:
* Make sure you have kubectl installed:
You can install kubectl according to the instructions in [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)

    * Download the latest release:
```shell
curl -LO https://storage.googleapis.com/kubernetes-release/release/latest/bin/windows/amd64/kubectl.exe
```
    * Add the binary in to your PATH.

    * Test to ensure the version of ```kubectl``` is the same as downloaded:
```shell
kubectl version
```
 I am use [chocolatey](https://chocolatey.org/) package manager for Windows. If you haven't it yet, go by link to install it. (also, you can install kubectl using choco, it's the easiest way)
 ```shell
 choco install minikube kubernetes-cli
 ```

### It's done, lets *start* minikube:

```shell
$ minikube start
* minikube v1.2.0 on windows (amd64)
* Downloading Minikube ISO ...
 129.33 MB / 129.33 MB  100.00% 0ss
* Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
* Configuring environment for Kubernetes v1.15.0 on Docker 18.09.6
* Downloading kubelet v1.15.0
* Downloading kubeadm v1.15.0
* Pulling images ...
* Launching Kubernetes ...
* Verifying: apiserver proxy etcd scheduler controller dns* Done! kubectl is now configured to use "minikube"
```
### Check that our minikube wm is going to be health:
```shell
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```




# Ingress
---
### Let's try to play with k8S ingress controller using minikube:

Ingress is a special "into" to your application. Smthng like wide window to outside world.
In K8s, Ingress works by configuring rules for accessing your applications.

Let's install ```ingress addon```:
```shell
$ minikube addons enable ingress
* ingress was successfully enabled
```

Check out our ingress-nginx running pod:
```shell
$ kubectl get pods -n kube-system | grep nginx-ingress-controller
nginx-ingress-controller-7b465d9cf8-x9hqc   1/1     Running   0          4m39s
```

# Deploy
---

1. Let's deploy our pods, using [deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/):
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: superapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: superapp
  template:
    metadata:
      labels:
        app: superapp
    spec:
      containers:
      - name: superapp
        image: gcr.io/kubernetes-e2e-test-images/echoserver:2.1
        ports:
        - containerPort: 8080
```
```shell
$ kubectl apply -f deployit.yaml
deployment.extensions/superapp created
```
2. Check that our pods are created:
```shel
$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
superapp-764569dc8-n56bg   1/1     Running   0          53s   172.17.0.8   minikube   <none>           <none>
superapp-764569dc8-r472x   1/1     Running   0          53s   172.17.0.9   minikube   <none>           <none>
```

3. Expose our pods using [services](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: superapp-svc
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: superapp
```

```shell
$ kubectl apply -f expose.yaml
service/superapp-svc created
$ kubectl get svc
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP   29m
superapp-svc   ClusterIP   10.99.235.97   <none>        80/TCP    10s
```
4. Setup the [Ingress Rules](https://kubernetes.io/docs/concepts/services-networking/ingress/):
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: superapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: \"false\"
spec:
  rules:
  - http:
      paths:
      - path: /meow
        backend:
          serviceName: superapp-svc
          servicePort: 80
```
```shell
$ kubectl apply -f ingress.yaml
ingress.extensions/superapp-ingress created
$ kubectl get ing
NAME               HOSTS   ADDRESS   PORTS   AGE
superapp-ingress   *                 80      19s
```

Check logs of our ing:
```shell
kubectl logs -n kube-system nginx-ingress-controller-7b465d9cf8-x9hqc
W0719 10:43:54.038106       6 flags.go:213] SSL certificate chain completion is disabled (--enable-ssl-chain-completion=false)
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:    0.23.0
  Build:      git-be1329b22
  Repository: https://github.com/kubernetes/ingress-nginx
-------------------------------------------------------------------------------

nginx version: nginx/1.15.9
W0719 10:43:54.049784       6 client_config.go:549] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0719 10:43:54.049960       6 main.go:200] Creating API client for https://10.96.0.1:443
I0719 10:43:54.069252       6 main.go:244] Running in Kubernetes cluster version v1.15 (v1.15.0) - git (clean) commit e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529 - platform linux/amd64
I0719 10:43:54.071935       6 main.go:102] Validated kube-system/default-http-backend as the default backend.
I0719 10:43:54.316709       6 nginx.go:261] Starting NGINX Ingress controller
I0719 10:43:54.326941       6 event.go:221] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"nginx-load-balancer-conf", UID:"e0c9b004-06d7-4e90-83b0-8637dcaed274", APIVersion:"v1", ResourceVersion:"713", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/nginx-load-balancer-conf
I0719 10:43:54.332573       6 event.go:221] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"tcp-services", UID:"28f9f339-61f7-4b79-bf33-e863739bef7e", APIVersion:"v1", ResourceVersion:"714", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/tcp-services
I0719 10:43:54.332603       6 event.go:221] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"udp-services", UID:"b94f6cb9-908d-498b-a78e-66e08bb0191b", APIVersion:"v1", ResourceVersion:"715", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/udp-services
I0719 10:43:55.519155       6 nginx.go:282] Starting NGINX process
I0719 10:43:55.519475       6 leaderelection.go:205] attempting to acquire leader lease  kube-system/ingress-controller-leader-nginx...
I0719 10:43:55.520430       6 controller.go:172] Configuration changes detected, backend reload required.
I0719 10:43:55.526365       6 leaderelection.go:214] successfully acquired lease kube-system/ingress-controller-leader-nginx
I0719 10:43:55.526681       6 status.go:148] new leader elected: nginx-ingress-controller-7b465d9cf8-x9hqc
I0719 10:43:55.665408       6 controller.go:190] Backend successfully reloaded.
I0719 10:43:55.665616       6 controller.go:200] Initial sync, sleeping for 1 second.
[19/Jul/2019:10:43:56 +0000]TCP200000.000
I0719 11:11:58.993938       6 event.go:221] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"superapp-ingress", UID:"a279a731-8065-467b-9dbe-c577729511b1", APIVersion:"extensions/v1beta1", ResourceVersion:"3172", FieldPath:""}): type: 'Normal' reason: 'CREATE' Ingress default/superapp-ingress
I0719 11:11:58.994088       6 controller.go:172] Configuration changes detected, backend reload required.
I0719 11:11:59.132221       6 controller.go:190] Backend successfully reloaded.
[19/Jul/2019:11:11:59 +0000]TCP200000.000
I0719 11:12:55.532417       6 status.go:388] updating Ingress default/superapp-ingress status from [] to [{10.0.2.15 }]
I0719 11:12:55.539196       6 event.go:221] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"superapp-ingress", UID:"a279a731-8065-467b-9dbe-c577729511b1", APIVersion:"extensions/v1beta1", ResourceVersion:"3250", FieldPath:""}): type: 'Normal' reason: 'UPDATE' Ingress default/superapp-ingress
192.168.99.1 - [192.168.99.1] - - [19/Jul/2019:11:13:33 +0000] "GET /meow HTTP/1.1" 308 171 "-" "curl/7.65.1" 82 0.000 [default-superapp-svc-80] - - - - d78a6337d4a204c473d17a5dc53b6ada
192.168.99.1 - [192.168.99.1] - - [19/Jul/2019:11:13:46 +0000] "GET /meow HTTP/1.1" 308 171 "-" "curl/7.65.1" 82 0.000 [default-superapp-svc-80] - - - - 2e8a97083acf9a781c690635a5fc1f89
```
