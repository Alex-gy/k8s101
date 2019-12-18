#部署HPA示例应用
kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --expose --port=80 --namespace dev

#设置CPU50%进行扩展,最大扩展为10个副本
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10 --namespace dev
在Cloud9环境中新开一个终端,启动压力客户端。通过脚本访问php-apache服务，持续增加压力。

#启动压力客户端
kubectl run -i --tty load-generator --image=busybox /bin/sh --namespace dev
#运行压力脚本 
while true; do wget -q -O - http://php-apache; done
