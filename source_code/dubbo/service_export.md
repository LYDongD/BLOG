## 服务暴露


### 定义

服务暴露即让服务可用

* 创建服务的URL
* 注册服务(远程暴露且采用注册中心时)
* 启动通信服务器


### 设计思想

* dubbo SPI 
    * 实现动态及延时扩展
* 装饰器模式
    * 结合spi扩展到自动包装机制，类似于aop
* 责任链模式
    * 在装饰器中加入监听器/过滤器等责任链进行协议暴露连续处理
* 动态代理
    * 动态创建自适应类
    * 动态创建服务引用的代理invoker

### 暴露流程

**1 服务暴露入口**

ServiceConfig


```

 public synchronized void export() {
        if (provider != null) {
            if (export == null) {
                export = provider.getExport();
            }
            if (delay == null) {
                delay = provider.getDelay();
            }
        }
        if (export != null && !export) {
            return;
        }

        //延迟暴露
        if (delay != null && delay > 0) {
            delayExportExecutor.schedule(new Runnable() {
                @Override
                public void run() {
                    doExport();
                }
            }, delay, TimeUnit.MILLISECONDS);
        } else {
            //具体暴露实现
            doExport();
        }
    }}


```

**要点分析**

* 配置优先级
    * 从配置来说，provider的优先级高于service，即provider的属性会覆盖service

* 延迟任务的实现方式
    * 使用调度线程池ScheduledExecutorService实现

```

//单后台线程执行
private static final ScheduledExecutorService delayExportExecutor = Executors.newSingleThreadScheduledExecutor(new NamedThreadFactory("DubboServiceDelayExporter", true));


//启动延时任务
if (delay != null && delay > 0) {
    delayExportExecutor.schedule(new Runnable() {
        @Override
        public void run() {
            doExport();
        }
    }, delay, TimeUnit.MILLISECONDS);
} 

```

* 关于延迟暴露的delay属性：

    * 默认：不设置delay或设置delay=-1, 即delay为null，默认等待容器完全初始化后才进行暴露
    * 设置delay=5000s具体时间，则在该service bean初始化后延迟指定时间后进行暴露

```

    //默认情况：delay=null 或 delay=-1时，等待容器完全初始化后暴露
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        if (isDelay() && !isExported() && !isUnexported()) {
            if (logger.isInfoEnabled()) {
                logger.info("The service ready on spring started. service: " + getInterface());
            }

            //向上引用父类ServiceConfig的暴露方法
            export();
        }
    }


   //service bean 初始化完后，进行暴露
   public void afterPropertiesSet() throws Exception {
      ....

      if (!isDelay()) {
          export();
      }
   }

   //是否完全delay，即等到容器完全初始化成功后才进行暴露
    private boolean isDelay() {
        Integer delay = getDelay();

        //provider的delay属性优先
        ProviderConfig provider = getProvider();
        if (delay == null && provider != null) {
            delay = provider.getDelay();
        }

        //用一个状态boolean来控制行为开关，该boolean被声明为transient
        return supportedApplicationListener && (delay == null || delay == -1);
    }

  
```

对于容器而言，执行顺序：

afterPropertiesSet -> onApplicationEvent

**2 暴露调用链**

export() -> doExport() -> doExportUrls() -> doExportUrlsFor1Protocol -> protocol.export(invoker)

```
  
    @SuppressWarnings({"unchecked", "rawtypes"})
    private void doExportUrls() {
        //加载注册中心(URL)列表： registry://host:port/interfaceName?application=xxx&dubbo=2.5.3&group=liam.....
        List<URL> registryURLs = loadRegistries(true);

        //向多个注册中心协议逐个暴露服务
        for (ProtocolConfig protocolConfig : protocols) {
            //使用单个应用协议暴露服务(本地暴露和远程暴露，本地暴露不会注册)
            doExportUrlsFor1Protocol(protocolConfig, registryURLs);
        }
    }


```

**核心暴露方法：**

* 采用默认协议dubbo

* 根据scope进行暴露
    * 本地暴露
		* 不需要注册
		* 不需要启动通信服务器，服务在jvm进程内调用

    * 远程暴露
		* 非直连模式需要注册
		* 需要启动通信服务器

    * 默认采用两种方式的暴露

* 协议暴露装饰器模式：
    * 从RegistryProtocol -> DubboProtocol，每个协议都被装饰了监听和过滤功能
    	* ProtocolListenerWrapper -> ProtocolFilterWrapper -> {RegistryProtocol,DubboProtocol}
			* ProtocolListenerWrapper暴露时创建监听器链
			* ProtocolFilterWrapper暴露时创建过滤器链
			* RegistryProtocol 实现服务注册后，又从头遍历了一遍协议暴露链，再到DubboProtocol
			* DubboProtocol 启动通信服务器


```

private void doExportUrlsFor1Protocol(ProticolConfig protocolConfig, List<URL> registryURLs){

    //默认采用dubbo应用协议进行暴露
    String name = protocolConfig.getName();
    if (name == null || name.length() == 0) {
        name = "dubbo";
    }

    ...

    //根据scope进行暴露
    String scope = url.getParameter(Constants.SCOPE_KEY);
    if (!Constants.SCOPE_NONE.equalsIgnoreCase(scope)) {
	//只要没有特别指定远程暴露，就一定会进行本地暴露
	if (!Constants.SCOPE_REMOTE.equalsIgnoreCase(scope)) {
                //本地暴露的url处理成：injvm://127.0.0.1/interfaceName？xxx
                exportLocal(url);
        }

	//只要没有特别指定本地暴露，就一定会进行远程暴露
	if (!Constants.SCOPE_LOCAL.equalsIgnoreCase(scope)) {

		//非直连模式，需要注册服务
                if (registryURLs != null && !registryURLs.isEmpty()) {
	            .....

                    for (URL registryURL : registryURLs) {

                        //使用 ProxyFactory 创建 Invoker 对象，持有服务引用的控制权，包含服务注册URL,其中服务暴露地址作为注册URL的参数
                        // registry://registry_host:port/com.alibaba.dubbo.RegistryService?application=xxx&export=dubbo://host:port/interfaceName?xxx
                        Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));

                        //在invoker的基础上增加了ServiceConfig
                        DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                        //使用 Protocol 暴露 Invoker 对象， 完成服务暴露
                        //此时的url是注册中心url, 自动识别为RegistryProtocol，在RegistryProtocol继续export服务的url
                        Exporter<?> exporter = protocol.export(wrapperInvoker);
                        exporters.add(exporter);
                    }
                } else { //无注册中心，不需要注册，服务直连模式(开发环境下可采用该模式，不需要注册中心，消费者需要明确知道提供者服务的URL，无法从注册中心获取)
                    Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, url);
                    DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                    Exporter<?> exporter = protocol.export(wrapperInvoker);
                    exporters.add(exporter);
                }
            }
	}
    }
}



```
