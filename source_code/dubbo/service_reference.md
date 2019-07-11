## 服务引用


### 默认情况下，什么时候开启本地引用？

**InjvmProtocol：**

```

if (getExporter(exporterMap, url) != null) {
    // by default, go through local reference if there's the service exposed locally
    isJvmRefer = true;
}

```

如果引用的服务进行了在该进程内本地暴露，则开启本地引用


### 服务引用做了哪些事情?

1. 向注册中心注册自己
2. 向注册中心订阅服务，获取服务提供者列表
3. 启动通信客户端

### 如何创建服务引用?

* 本地引用: ReferenceConfig -> InjvmInvoker
* 远程引用: ReferenceConfig -> DubboInvoker

* 引用过程：

1. 多协议处理：依赖协议自适应机制
2. 添加责任链: filters 和 listeners 


### 通信模块客户端有几种连接模式？

1 缺省: 单一长连接，所有客户端共享一个连接
2 为引用指定连接数: 该引用独享1个或多个连接通道

```
<dubbo:reference interface="com.foo.BarService" connections="10" />

```

dubbo通信模块采用netty或mima实现，采用NIO实现socket io
