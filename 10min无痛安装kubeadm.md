## centos 7 安装kubeadm1.16.3

### 一、安装docker-ce
#### 安装依赖
```
sudo yum update -y && sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

#### 安装docker
```
curl -fsSL "https://get.docker.com/" | bash -s -- --mirror Aliyun && yum autoremove docker-ce -y
yum install -y docker-ce-18.06.1.ce-3.el7
systemctl enable docker && systemctl start docker
```

#### 修改docker cgroup驱动并增加国内加速源

```
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://registry.docker-cn.com"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
```
#### 重启使配置生效 & 开机自启
```
systemctl restart docker && systemctl enable --now docker
```

### 二、安装kubeadm

#### 设置国内阿里源
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
#### 关闭SElinux& 防火墙
```
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
systemctl disable --now firewalld
```
#### centos7用户还需要设置路由：
```
yum install -y bridge-utils.x86_64
```
#### 加载br_netfilter模块，使用lsmod查看开启的模块
```
modprobe  br_netfilter
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

#### k8s要求关闭swap 
```
swapoff -a && sysctl -w vm.swappiness=0  # 关闭swap
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab  # 取消开机挂载swap
```
#### 安装kubelet kubeadm kubectl
```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

#### 开机启动kubelet
```
systemctl enable --now kubelet
```

### 三、使用kubeadm创建集群

#### 初始化master节点
```
kubeadm init \
--image-repository registry.aliyuncs.com/google_containers \
--pod-network-cidr=10.244.0.0/16 \
--ignore-preflight-errors=cri \
--kubernetes-version=1.16.3
```

#### 设置权限
```
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### pod显示pending（需要安装网络插件）
```
[root@192-168-1-163 ~]# kubectl get pod -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-58cc8c89f4-5856l                0/1     Pending   0          102s
coredns-58cc8c89f4-hkwpt                0/1     Pending   0          102s
etcd-192-168-1-163                      1/1     Running   0          46s
kube-apiserver-192-168-1-163            1/1     Running   0          56s
kube-controller-manager-192-168-1-163   1/1     Running   0          57s
kube-proxy-ktzdr                        1/1     Running   0          101s
kube-scheduler-192-168-1-163            1/1     Running   0          45s
```
#### 网络插件(flannel方案）
```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### 修改镜像（指定国内镜像地址）
```
sed 's/quay.io\/coreos/registry.cn-beijing.aliyuncs.com\/imcto/g' kube-flannel.yml >> 1.yml
```

#### 安装网络插件
```
kubectl apply -f 1.yml
```

#### 网络插件（calico方案可选）
```
kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml
```

# 确保所有的Pod都处于Running状态。
kubectl get pod –all-namespaces -o wide

# master node参与工作负载（如果单节点，创建pod会处于pedding状态）
查看污点标记
kubectl describe node |grep Taint
Taints:             node-role.kubernetes.io/master:NoSchedule
执行命令去除标记
kubectl taint nodes --all node-role.kubernetes.io/master-


# 多台节点加入（单节点可跳过）
kubeadm join 192.168.1.163:6443 --token 5moxhb.40obnw7uoz6dy506 \
    --discovery-token-ca-cert-hash sha256:90d59ad0504c1291166f55bb0dc980fc87f8fc3760d23df141858962614d5116

# 命令补全
yum install -y bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc


# 排错
journalctl -f  # 当前输出日志
journalctl -f -u kubelet  # 只看当前的kubelet进程日志

# 重置集群
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/

# kubeadm 模式开启ipvs转发（可选）
由于ipvs已经加入到了内核的主干，所以为kube-proxy开启ipvs的前提需要加载以下的内核模块：

ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack_ipv4

# 加载模块
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4

# 安装ipset软件包和ipvsadm
yum install ipset -y
yum install ipvsadm -y

# kube-proxy开启ipvs
修改ConfigMap的kube-system/kube-proxy中的config.conf，把 mode: "" 改为mode: “ipvs" 保存退出即可
kubectl edit cm kube-proxy -n kube-system

# 删除之前的proxy pod
kubectl get pod -n kube-system |grep kube-proxy |awk '{system("kubectl delete pod "$1" -n kube-system")}'

# 查看proxy运行状态
kubectl get pod -n kube-system | grep kube-proxy

# 查看日志,如果有 `Using ipvs Proxier.` 说明kube-proxy的ipvs 开启成功!
[root@192-168-1-163 ~]# kubectl logs kube-proxy-lz5cj -n kube-system
I1115 15:59:06.739713       1 node.go:135] Successfully retrieved node IP: 192.168.1.163
I1115 15:59:06.739779       1 server_others.go:176] Using ipvs Proxier.
W1115 15:59:06.740027       1 proxier.go:420] IPVS scheduler not specified, use rr by default
I1115 15:59:06.740320       1 server.go:529] Version: v1.16.3
I1115 15:59:06.740881       1 conntrack.go:52] Setting nf_conntrack_max to 131072
I1115 15:59:06.741163       1 config.go:131] Starting endpoints config controller
I1115 15:59:06.741201       1 shared_informer.go:197] Waiting for caches to sync for endpoints config
I1115 15:59:06.741235       1 config.go:313] Starting service config controller
I1115 15:59:06.741244       1 shared_informer.go:197] Waiting for caches to sync for service config
I1115 15:59:06.841382       1 shared_informer.go:204] Caches are synced for endpoints config
I1115 15:59:06.841391       1 shared_informer.go:204] Caches are synced for service config


# 安装dashboard
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta5/aio/deploy/recommended.yaml -O dashboard.yaml

vim dashboard.yaml
---
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort      #使用NodePort方式暴露service
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30443      #指定nodePort=30443
  selector:
    k8s-app: kubernetes-dashboard
---

# 部署
kubectl apply -f dashboard.yaml

为Dashboard创建 Service Account 和 ClusterRoleBinding
vi auth.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

# 执行rbac
kubectl apply -f auth.yaml

# 获取访问Dashboard的token
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

# 访问IP+端口号
需要用火狐浏览器打开
