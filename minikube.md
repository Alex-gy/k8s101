
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
