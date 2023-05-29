
## Pod对象

### Pod基本概念
![](assets/markdown-img-paste-20230528235321996.png)

Pod是一个逻辑抽象概念， Kubernetes创建和管理的最小单元，一个Pod由一个容器或多个容器组成。

Pod特点： 
* 一个Pod可以理解为是一个应用实例，提供服务  
* Pod中容器始终部署在一个Node上  
* Pod中容器共享网络、存储资源   

### Pod存在的意义
Pod主要用法：
* 运行单个容器： 最常见的用法，在这种情况下，可以将Pod看做是单个容器的抽象封装  
* 运行多个容器： 属于边车模式（Sidecar） ，通过再Pod中定义专门容器，来执行主业务容器需要的辅助工作，这样好处是将辅助功能同主业务容器解耦，实现独立发布和能力重用。  
  - 例如：
    - 日志收集
    - 应用监控


边车模式类似于生活中边三轮摩托车，通过摩托车上增加一个边车方式来扩展现有的服务和功能
![](assets/markdown-img-paste-20230529203748302.png)

### Pod管理命令
定义Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: container1
    image: nginx
  - name: container2
    image: centos
```

Pod管理命令:
```sh
创建Pod：
kubectl apply -f pod.yaml
或者使用命令： kubectl run nginx --image=nginx
查看Pod：
kubectl get pods
kubectl describe pod <Pod名称>
查看日志：
kubectl logs <Pod名称> [-c CONTAINER]
kubectl logs <Pod名称> [-c CONTAINER] -f
进入容器终端：
kubectl exec <Pod名称> [-c CONTAINER] -- bash
删除Pod：
kubectl delete pod <Pod名称>
```









