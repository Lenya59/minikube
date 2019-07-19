# Install Minikube
This page shows you how to install Minikube, a tool that runs a single-node Kubernetes cluster in a virtual machine on your personal computer.

## Make sure you have kubectl installed.
You can install kubectl according to the instructions in [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)

* Download the latest release
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

It's done, lets start(```minikube start```) minikube:

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

## Check the status

You  can see the status of Minikube using the ```minikube status``` command:
```shell
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```
