安装一台虚拟机
multipass launch --name k8s --mem 2G --disk 20G

multipass exec k8s -- sudo iptables -P FORWARD ACCEPT

进入虚拟机
multipass shell k8s

查询microk8s
snap info microk8s

安装microk8s
snap install microk8s --classic

开启组件（自定义选择）
microk8s.enable dashboard dns ingress istio registry storage

查看排障
microk8s.inspect

别名
snap alias microk8s.kubectl kubectl

命令补全
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
