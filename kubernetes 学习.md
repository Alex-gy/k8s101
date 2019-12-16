# kubernetes学习

## 资料篇

### 文档资料：
官方文档
    https://kubernetes.io/docs/concepts/

宋净超 Kubernetes中文指南/云原生应用架构实践手册
    https://jimmysong.io/kubernetes-handbook/

feisky Kubernetes指南
    https://kubernetes.feisky.xyz/

张磊 深入剖析Kubernetes（付费专栏，有电子离线版）
    https://time.geekbang.org/column/116


视频资料：
马哥 Kubernetes（K8s）从入门到精通
    https://www.bilibili.com/video/av44264629?from=search&seid=14189051786765129632

华为云 CKA认证
    https://www.bilibili.com/video/av46687897?from=search&seid=11011730165026290716

阿里云 云原生技术公开课
    https://edu.aliyun.com/roadmap/cloudnative?spm=5176.8764728.631162.110.60b920beC8hNRY


图书资料：
    kubernetes action（有pdf电子版）
    kubernetes 进阶实战（微信读书免费）
    kubernetes 权威指南第四版（微信读书免费）


学习路径：
才云 Kubernetes 学习路径
    https://zhuanlan.zhihu.com/p/81565088

微软 Kubernetes Learning Path v2.0（英文）
     https://azure.microsoft.com/en-us/resources/kubernetes-learning-path/



实践篇

云上实验环境：
katacoda
    https://katacoda.com/（推荐）

谷歌官方的play with k8s（翻墙）
    https://labs.play-with-k8s.com/#
    

搭建环境：

二进制搭建
    https://github.com/kubernetes/kubernetes
    go语言程序，部署相对容易，难点在于证书管理/etcd/master节点高可用，不建议初学者用此法直接搭建。

kubeadm搭建
    官方推荐实验环境，需要指定软件和镜像源在国内地址，可以看我写的安装总结
    https://github.com/Alex-gy/k8s101/blob/master/10min%E6%97%A0%E7%97%9B%E5%AE%89%E8%A3%85kubeadm.md

mac os和window10 版本docker desktop自带的k8s
    https://www.docker.com/products/docker-desktop
    比较简单，易于上手（推荐）

第三方ansbile安装脚本
    https://sealyun.com/docs/
    网上有很多，可以尝试下
