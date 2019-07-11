## dubbo源码阅读 - 项目准备

> #### dubbo-demo 模块

```
| dubbo-demo
	| dubbo-demo-api
	| dubbo-demo-consumer
	| dubbo-demo-producer
```

从模块名称来看，这里包含了rpc最核心的3个角色：

* consumer 服务消费者
* provider 服务提供者
* api 服务接口

**在14h前，该模块被更新了，提交日志：**

```

//将基于xml配置bean的dubbo demo转移至相关目录
* move dubbo xml demo into delegated dir

//增加注解使用demo，将dubbo-demo-api更名为dubbo-demo-interface
* add demo for annotation usage, and rename dubbo-demo-api to dubbo-demo-interface

//增加基于api的demo
* add pure api demo

//提供更强大的用例
* enhance examples

//移除一些不必要的导入
* remove useless imports

//给模块增加readme
* add readme for dubbo-demo

```

综上所述，**新版本增加了基于注解和基于api的使用方式**

---


### 准备调试环境


以xml demo 为例, 实现DemoService的RPC调用：

```
| dubbo-demo-xml
	| dubbo-demo-xml-consumer
		| Application
	| dubbo-demo-xml-provider
		| Applicaiton
		| DemoServiceImpl
		
| dubbo-demo-interface
	| DemoService
	
```


> #### 启动provider

1. 读取spring配置文件，装载bean
2. 启动容器
3. 阻塞主进程 

```
## 问题1： multicast 启动报错“Can't assign requested address”

启动jvm时添加参数(vm option)：

-Djava.net.preferIPv4Stack=true
```
> #### 启动consumer

1. 读取spring配置文件，装载bean
2. 启动容器
3. 从容器中获取指定service bean
4. 调用bean的方法










