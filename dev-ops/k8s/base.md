## kubernetes 入门

### 功能

* 自动部署，重启和伸缩
* 资源隔离
* 监看检测
* 滚动更新
* 服务自动发现
* 负载均衡

### 架构

* master
    * etcd
    * api server
    * controller
* nodes
    * namespace
    * label/pod/replicas/deployment
    * service（lable selector)
        * pods load balance
    * ingress(nginx

client -> ingress -> service -> node -> pod

pod -> node -> flannel -> node -> pod
* images repository



### k8s标准定义

* 资源分配定义
* 服务标准定义

