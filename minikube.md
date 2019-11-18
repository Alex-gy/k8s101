### 安装kubectl
可以从kubernetes库上直接下载，方法如下：

step 1: 访问官方github网址：https://github.com/kubernetes/kubernetes/releases

step 2: 找到想使用的发布版本，在每个发布版本的最后一行有类似“CHANGELOG-1.10.md”这样的内容，点击超链进入；

step 3: 然后进入“Client Binaries”区域；

step 4: 选择和目标机器系统匹配的二进制包下载；

step 5: 解压缩，放入/usr/local/bin目录；

### 安装minikube
首先记住阿里云发布的minikube地址：https://github.com/AliyunContainerService/minikube

从release目录下载最新的minikube版本，然后：

`chmod +x minikube
mv minikube /usr/local/bin`



### 自定义配置
```
minikube start --image-mirror-country cn \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.5.1.iso \
    --registry-mirror=https://6iyi5l11.mirror.aliyuncs.com \
    --cpus=2 \
    --memory=2000mb \
    --disk-size=5g
```


### 使用containerd作为引擎
```
minikube start --image-mirror-country cn \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.5.1.iso \
    --registry-mirror=https://6iyi5l11.mirror.aliyuncs.com \
    --container-runtime=containerd
```

### 创建pod
```
kubectl run nginx --image=nginx --restart=Never
```
### 进入pod查看内核,发现和node节点同一内核（废话）
```
➜  ~ kubectl exec nginx -- uname -a                 
Linux nginx 4.19.76 #1 SMP Tue Oct 29 14:56:42 PDT 2019 x86_64 GNU/Linux
➜  ~ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ uname -a
Linux minikube 4.19.76 #1 SMP Tue Oct 29 14:56:42 PDT 2019 x86_64 GNU/Linux

```
### 开启对gvisor支持
```
➜  ~ minikube addons enable gvisor
✅  gvisor was successfully enabled
➜  ~ kubectl get pod,runtimeclass gvisor -n kube-system
NAME         READY   STATUS    RESTARTS   AGE
pod/gvisor   1/1     Running   0          33s

NAME                              CREATED AT
runtimeclass.node.k8s.io/gvisor   2019-11-18T04:19:29Z
➜  ~ kubectl get runtimeclasses.node.k8s.io gvisor     
NAME     CREATED AT
gvisor   2019-11-18T04:19:29Z
```
### 创建一个运行在 gvisor 沙箱容器中的 nginx 应用。
```
cat nginx-untrusted.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-untrusted
spec:
  runtimeClassName: gvisor
  containers:
  - name: nginx
    image: nginx
$ kubectl apply -f nginx-untrusted.yaml
pod/nginx-untrusted created
$ kubectl exec nginx-untrusted -- uname -a
Linux nginx-untrusted 4.4 #1 SMP Sun Jan 10 15:06:54 PST 2016 x86_64 GNU/Linux
```
> 我们可以清楚地发现：由于基于 runc 的容器与宿主机共享操作系统内核，runc 容器中查看到的 OS 内核版本与 Minikube 宿主机 OS 内核版本相同；而 gvisor 的 runsc 容器采用了独立内核，它和 Minikube 宿主机 OS 内核版本不同。正是因为每个沙箱容器拥有独立的内核，减小了安全攻击面，具备更好的安全隔离特性。适合隔离不可信的应用，或者多租户场景。

### 使用 ctl 和 crictl 工具


我们现在可以进入进入 Minikube 虚拟机：
 ```
$ minikube ssh

containerd 支持通过名空间对容器资源进行隔离，查看现有 containerd 名空间：

$ sudo ctr namespaces ls
NAME   LABELS
k8s.io
```
### 列出所有容器镜像
`$ sudo ctr --namespace=k8s.io images ls`
...
### 列出所有容器列表
`$ sudo ctr --namespace=k8s.io containers ls`

在 Kubernetes 环境更加简单的方式是利用 crictl 对 pods 进行操作。

### 查看pod列表
`$ sudo crictl pods
POD ID              CREATED             STATE               NAME                                         NAMESPACE              ATTEMPT
78bd560a70327       3 hours ago         Ready               nginx-untrusted                              default                0
94817393744fd       3 hours ago         Ready               nginx                                        default                0
...
`
# 查看名称包含nginx的pod的详细信息
```
$ sudo crictl pods --name nginx -v
ID: 78bd560a70327f14077c441aa40da7e7ad52835100795a0fa9e5668f41760288
Name: nginx-untrusted
UID: dda218b1-d72e-4028-909d-55674fd99ea0
Namespace: default
Status: Ready
Created: 2019-10-27 02:40:02.660884453 +0000 UTC
Labels:
    io.kubernetes.pod.name -> nginx-untrusted
    io.kubernetes.pod.namespace -> default
    io.kubernetes.pod.uid -> dda218b1-d72e-4028-909d-55674fd99ea0
Annotations:
    kubectl.kubernetes.io/last-applied-configuration -> {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"nginx-untrusted","namespace":"default"},"spec":{"containers":[{"image":"nginx","name":"nginx"}],"runtimeClassName":"gvisor"}}
    kubernetes.io/config.seen -> 2019-10-27T02:40:00.675588392Z
    kubernetes.io/config.source -> api
ID: 94817393744fd18b72212a00132a61c6cc08e031afe7b5295edafd3518032f9f
Name: nginx
UID: bfcf51de-c921-4a9a-a60a-09faab1906c4
Namespace: default
Status: Ready
Created: 2019-10-27 02:38:19.724289298 +0000 UTC
Labels:
    io.kubernetes.pod.name -> nginx
    io.kubernetes.pod.namespace -> default
    io.kubernetes.pod.uid -> bfcf51de-c921-4a9a-a60a-09faab1906c4
Annotations:
    kubectl.kubernetes.io/last-applied-configuration -> {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"nginx","namespace":"default"},"spec":{"containers":[{"image":"nginx","name":"nginx"}]}}
    kubernetes.io/config.seen -> 2019-10-27T02:38:18.206096389Z
    kubernetes.io/config.source -> api
```

