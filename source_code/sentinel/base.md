## sentinel应用场景及限流算法原理

### 场景

* 流量暴增，例如双11，柜机系统升级数据同步(10w+)等，流量暴增导致服务过载。
* 下游服务崩溃或时好时坏，例如登录服务依赖用户画像服务，下游服务宕机导致快递员无法登陆。

### 解决方案

* 流量限制
* 服务熔断，降级

### 问题

1 如何量化的描述流量？怎么判断流量暴增？

* QPS， 1s内的请求数，如果QPS超过日常平均值，则认为是流量过高。例如平时QPS为100，突然增加到1000。
* 并发线程数，通常指的是为业务分配的线程池最大线程数，如果线程池在一段时间内持续爆满，则认为是流量暴增。例如tomcat线程池默认最大线程数200，但是没有可用线程。

2 如何量化描述下游服务的状态？

* RT，请求响应延时，如果下游出现问题，通常RT会超过平均值。例如RT为100ms的请求，下游出现问题时，RT超过2s，甚至超时。
* 异常比例，即一段时间内异常请求的占比，如果下游不稳，请求失败抛出异常，一段时间内异常比例上升。例如上升到50%以上，平均每2次请求会失败1个。

综上，解决方案中，流量限制，可以根据QPS或并发线程数；服务熔断降级，则可以根据RT或异常比例，

### 方案设计

通过场景问题分析，我们只需要做2件事情即可：

1. 收集并统计以上量化指标，每次请求都计算QPS,RT,并发线程数和异常比例。
2. 根据量化指标和设置的规则在请求前决定是否进行流控或熔断降级。

可以通过AOP或拦截器机制，在方法调用前进行以上两件事情的处理。

[参考设计方案](https://www.processon.com/diagraming/5d5e50bae4b04e664f2e316d)

### 开源方案

阿里开源的sentinel已经为我们实现了以上功能。

[参考官方文档](https://github.com/alibaba/Sentinel/wiki/Sentinel%E5%B7%A5%E4%BD%9C%E4%B8%BB%E6%B5%81%E7%A8%8B)

* 拦截器链： Slot Chain
* 通过Slot完成对应的拦截处理，如统计，限流，熔断等。

#### sentinel是如何实现拦截器链的？

**基于链表的责任链设计模式实现拦截器链**

sentinel为每个功能定义一个slot节点，并用责任链模式将这些节点串联起来。责任链作为切面拦截器，在目标方法调用前后进行拦截调用，依次遍历每个slot节点进行调用。


> 如何生成责任链？

* DefaultSlotChainBuilder#build 单向链表生成算法
* CtSph#lookProcessChain 获取或生成责任链，为每个资源生成一条链

> 如何调用责任链？

研究方法的进和出分别是怎么调用责任链的

**sentinel用Entry管理方法的入和出**

* CtSph#entryWithPriority -> chain.entry()

从第一个节点(first)开始，依次调用节点的entry方法

1. 调用第一个节点的entry方法
2. 第一个节点是哨兵节点，什么都不做, 直接调用fireEntry方法
3. fireEntry将请求传递到下一个节点
4. 调用第二个节点的entry方法，重复1，2，3

* CtSph#exitForContext -> chain.exit()

从第一个节点(first)开始，依次调用节点的exit方法

1. 调用第一个节点的entry方法
2. 第一个节点是哨兵节点，什么都不做, 直接调用fireExit方法
3. fireExit将请求传递到下一个节点
4. 调用第二个节点的exit方法，重复1，2，3

> 总结

```
1 入口，会先获取责任链，没有则为该资源创建一个。
2 入和出责任链调用顺序是一致的，都是从first -> end， 所以单向链表是足够的。
3 SlotChainBuilder是一个spi接口，可自定义slot，构建自己的责任链，例如想根据时间段进行限流。
```

### 其他框架的责任链模式实现(开放讨论）

> tomcat

pipline-Vale 责任链机制，也是用单向链表实现


----

**TODO:**


#### 如何计算QPS，RT等指标？

研究 StatisticSlot 组件的实现

StatisticSlot 定义了Metric对象进行统计，在方法退出时，将RT, QPS(区分正常，异常QPS）添加到Metric

这里，用于统计的数据结构是 LeapArray<MetricBucket>




#### 流量限制有几种策略? 分别是如何实现的？

研究 FlowSlot 组件的实现 

##### 流控的策略通常有以下3种：

* 匀速排队(漏铜算法)
* 直接拒绝
* Warm Up











