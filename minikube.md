
### 使用containerd作为引擎
```
minikube start --image-mirror-country cn \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.5.1.iso \
    --registry-mirror=https://6iyi5l11.mirror.aliyuncs.com \
    --container-runtime=containerd
```

### 自定义配置
```
minikube start --image-mirror-country cn \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.5.1.iso \
    --cpus=2 \
    --memory=2000mb \
    --disk-size=3g
```
