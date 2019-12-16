### 安装

#### centos7前置检查
https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh

#### 前置知识
Docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版），我们用社区版就可以了

#### 安装

`export VERSION=19.03
`
`
curl -fsSL "https://get.docker.com/" | bash -s -- --mirror Aliyun
`
#### 配置启动参数
```
mkdir -p /etc/docker/
cat>/etc/docker/daemon.json<<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": [
      "https://fz5yth0r.mirror.aliyuncs.com",
      "http://hub-mirror.c.163.com/",
      "https://docker.mirrors.ustc.edu.cn/",
      "https://registry.docker-cn.com"
  ],
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
EOF
```
#### 补全docker命令
`
yum install -y epel-release bash-completion
`

#### 重启使配置生效 & 开机自启
`
systemctl restart docker && systemctl enable --now docker
`
#### 验证
`
docker info  & docker version
`
