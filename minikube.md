
### 自定义配置
```
minikube start --image-mirror-country cn \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.5.1.iso \
    --cpus=2 \
    --memory=2000mb \
    --disk-size=3g
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

