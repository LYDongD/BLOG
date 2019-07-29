## springboot 启动

### 流程

```
public class CabinetQueryServerApplication {
	public static void main(String[] args) {
		SpringApplication.run(CabinetQueryServerApplication.class, args);
	}
}

```

应用的启动分成两步：

* 创建SpringApplication
* 调用run方法启动

```
	public static ConfigurableApplicationContext run(Class<?>[] primarySources,
			String[] args) {
		return new SpringApplication(primarySources).run(args);
	}

```

### run

仅保留核心逻辑

```
public ConfigurableApplicationContext run(String... args) {
       

		ConfigurableApplicationContext context = null;
        //获取监听器并通知容器正在启动
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
            //配置参数和环境，包括加载application.properties属性文件的所有属性
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
			configureIgnoreBeanInfo(environment);

            //创建spring容器
			context = createApplicationContext();

            //容器上下文初始化前置处理 -> 初始化 -> 初始化后置处理
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);

            //通知监听器容器已启动
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
            //通知，容器正在运行
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

这里的主要逻辑为：

* 创建并初始化容器
* 通知监听器


> 事件通知机制


![事件通知机制](http://blogimage.ponymew.com/liam/springboot_listeners.png)


这里比较有代表性的两个事件

* ContextRefreshedEvent, 在容器refreshContext初始完后发布
* ApplicationStartedEvent，在容器完全启动成功后发布

从顺序上看，ApplicationStartedEvent在ContextRefreshedEvent之后

关于spring事件发布可参考另一篇文章[事件发布](http://www.ponymew.com/source_code/spring/event.html)


> spi机制

这里的SpringApplicationRunListener只有一个实现类，我们无法扩展实现该类

该实现类通过spi机制加载，即创建SpringApplication的时候从META-INF/spring.factries加载接口的实现类

```
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener

```
