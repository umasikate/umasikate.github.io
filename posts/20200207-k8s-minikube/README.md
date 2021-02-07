
# Description

Env is MacOS

We are going to learn how to use k8s with minikube

I wanted to use 3 or 4 Raspberry Pis for the sake of better understanding such as Cluster/Node, but I have no such money. haha
(If you use them, please check what is called `Kubeadm`)
(In the case of Cloud Service, please check `Google GKE/Azure AKS/AWS EKS`)

Detail about the application that we use this time

- golang (web server)
- db

# Short Explanation

these might be wrong

- Node
  - something like host
- Cluster
  - typically has one or more nodes
- Pod
  - minimum deployable unit
- Service
  - abstract service related to internet connection
- kubectl
  - for cli tool
- minikube
  - single node k8s local testing tool


# Preparation

- Virtual Box

is needed

```bash
 $ brew install kubectl
 $ kubectl version --client
 $ sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
 $ brew install minikube
 $ minikube --version
 $ minikube start --driver=virtualbox
 $ minikube status
```

Staring Minukube failed, I dealt with it.

[kubernetes - How to fix VM issue with minikube start ? - Stack Overflow](https://stackoverflow.com/questions/52277019/how-to-fix-vm-issue-with-minikube-start)

```bash
$ minikube start --driver=virtualbox
😄  minikube v1.16.0 on Darwin 11.1
✨  Using the virtualbox driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🔥  Creating virtualbox VM (CPUs=2, Memory=4000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.20.0 on Docker 20.10.0 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
timeToStop: Nonexistent
```

Looks good


```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://xxx.xxx.xx.xxx:xxxx
KubeDNS is running at https://xxx.xxx.xx.xxx:xxxx/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
$ kubectl get nodes
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   2m11s   v1.20.0

$ minikube dashboard
🔌  Enabling dashboard ...
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening
```


# Create a Pod

From Official Documentation

> Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

Before I get into the Pod explanation, let's play with a simplest nginx example and grasp the Pod picture


```bash
$ kubectl run nginx-pod --image=nginx
pod/nginx-pod created

$ kubectl get pods
NAME        READY   STATUS              RESTARTS   AGE
nginx-pod   0/1     ContainerCreating   0          13s

$ kubectl port-forward nginx-pod 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080

$ curl http://localhost:8080

$ kubectl delete pods nginx-pod
pod "nginx-pod" deleted
```


Then, Pod has following features

- Has one or more containers
- Shares networks and storages
- Be Deployed in the same Node


Above example can be written with `yaml` file

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
```


Let's deploy it again


```bash
$ kubectl create -f 1_pod.yaml
configmap/init-db-sql created
pod/sample-pod created
service/flexy-demo-all-in-one created
```


Nexa, deploy a more practical example

![](https://flxy.jp/wp-content/uploads/2020/02/Kubernetes02.png)


```bash
$ kubectl apply -f 1_pod.yaml
configmap/init-db-sql created
pod/sample-pod created
service/flexy-demo-all-in-one created

$ kubectl get services
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
flexy-demo-all-in-one   LoadBalancer   xx.xxx.xxx.xxx   <pending>     8080:32687/TCP   25s
kubernetes              ClusterIP      xx.xx.x.x        <none>        443/TCP          42m

$ kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
sample-pod   2/2     Running   0          44s

$ kubectl delete -f 1_pod.yaml
configmap "init-db-sql" deleted
pod "sample-pod" deleted
service "flexy-demo-all-in-one" deleted
```

Pod has two containers
Besides, there is loadbalancer, whereby we can access it from the Internet (Grobal IP)


```bash
$ minikube stop
✋  Stopping node "minikube"  ...
🛑  1 nodes stopped.
```


# References

- [Kubernetes実践入門。基本的なyamlとコマンドから学ぶサービス運用効率化術 \| FLEXY（フレキシー）](https://flxy.jp/article/10107)
- [visual studio code - One or more containers do not have resource limits (Kubernetes) warning in vscode kubernetes tools - Stack Overflow](https://stackoverflow.com/questions/64080471/one-or-more-containers-do-not-have-resource-limits-kubernetes-warning-in-vscod)
- [Kubernetesのマニフェスト（Manifest）ファイルでNginxを起動](https://noumenon-th.net/programming/2019/04/16/manifest01/)
- [Kubernetesの基礎 \| Think IT（シンクイット）](https://thinkit.co.jp/article/13542)
- [GitHub - MasayaAoyama/flexy-demo](https://github.com/MasayaAoyama/flexy-demo)

