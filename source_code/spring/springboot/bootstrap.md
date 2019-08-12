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

> 容器初始化流程

* 构造应用上下文环境
* 初始化应用上下文
* 刷新应用上下文前的准备阶段
* 刷新应用上下文
* 刷新应用上下文后的扩展接口

准备阶段【ConfigurationClassParser】: springboot会优先加载主类，注册到容器(DefaultListableBeanFactory)


刷新阶段: 解析主类的注解，进一步进行容器初始化

* 资源定位(主类包，@ComponentScan和starter的spring.factories)
* 资源加载(BeanDefinition)
* 资源注册(BeanDefinition)
* 依赖注入(创建bean并递归完成注入)


[参考](https://juejin.im/post/5ca42bfa6fb9a05e17799e07)

### 应用启动异常报告机制

![异常报告](http://blogimage.ponymew.com/liam/异常处理.png)

如图所示，springboot应用启动时，如果发送异常，会通过SpringBootExceptionReporter将异常报告给用户

FailureAnalyzers 整合了分析器FailureAnalyzer和报告器FailureAnalysisReporter列表，轮询所有通过spi机制加载的组件，如果组件可以进行处理则进行处理

* 通过FailureAnalyzer对异常(Throwable)进行分析，生成分析结果FailureAnalysis
* 异常分析结果FailureAnalysis包含三部分
    * description 异常描述
    * action 异常解决方案
    * cause 异常本身
* 通过FailureAnalysisReporter将分析结果报告出去，例如LoggingFailureAnalysisReporter将分析结果以指定格式打印


例如，应用启动时，如果tomcat指定的端口被其占用，最终会打印：

```
***************************
APPLICATION FAILED TO START
***************************

Description:

The Tomcat connector configured to listen on port 8080 failed to start. The port may already be in use or the connector may be misconfigured.

Action:

Verify the connector's configuration, identify and stop any process that's listening on port 8080, or configure this application to listen on another port.

```

这里的异常为：

ConnectorStartFailedException

最终由LoggingFailureAnalysisReporter实现异常报告，即打印。
