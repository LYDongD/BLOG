## 配置

#### API配置

* 模块: dubbo-config-api


> 配置分类

```
1 消费者配置
2 提供者配置
3 应用配置
4 其他配置

```

> 配置方式

```
1 api
2 xml
3 系统属性/环境变量

```

> 配置使用

在进行服务暴露或服务引用前，需要加载特定的配置，所有的配置最终转化为dubbo URL

**服务暴露：**

```
public class ProviderApplication {


    public static void main(String args[]) throws Exception{

        //api方式添加配置，并暴露服务
        ServiceConfig<DemoServiceImpl> serviceConfig = new ServiceConfig();
        serviceConfig.setApplication(new ApplicationConfig("dubbo-demo-provider"));
        serviceConfig.setRegistry(new RegistryConfig("multicast://224.5.6.7:1234"));
        serviceConfig.setInterface(DemoService.class);
        serviceConfig.setRef(new DemoServiceImpl());
        serviceConfig.export();
        System.in.read();
    }
}

```

**服务引用：**

```
public class ConsumerApplication {

    public static void main(String args[]) {
        //通过api方式添加配置，并获取一个服务引用
        ReferenceConfig<DemoService> referenceConfig = new ReferenceConfig<>();
        referenceConfig.setApplication(new ApplicationConfig("dubbo-demo-cosumer"));
        referenceConfig.setRegistry(new RegistryConfig("multicast://224.5.6.7:1234"));
        referenceConfig.setInterface(DemoService.class);
        DemoService demoService = referenceConfig.get();

        //rpc调用
        String name = demoService.sayHello("liam");
        System.out.println("rpc call success: " + name);

    }
}

```

> 配置源码解读

**1 如何获取配置对象的属性？**

反射机制

* 获取配置对象
* 获取对象的所有方法
* 解析合法的get方法
* 解析方法名获取属性名
* 动态调用get方法获取属性值

**2 拼接参数和拼接属性有什么区别？**

* 都是将配置对象的属性添加到参数表当中

查找方法被调用点，dubbo项目内只有测试类使用了appendAttribute,而ServiceConfig,ReferenceConfig这些配置类主要使用了appendParameter


**3 完整的服务URL是什么样的？**

* protocol://username:password@host:port/path?key=value&key=value

```
dubbo://192.168.3.17:20880/com.alibaba.dubbo.demo.DemoService?anyhost=true&application=demo-provider&default.delay=-1&default.retries=0&default.service.filter=demoFilter&delay=-1&dubbo=2.0.0&generic=false&interface=com.alibaba.dubbo.demo.DemoService&methods=sayHello&pid=19031&side=provider&timestamp=1519651641799

```

**4 配置中的前缀prefix是什么？**

例如default, 表示默认属性

**5 配置对象那些属性被添加了注解@parameter？**

ServiceConfig:

```
//exclued=true排除了该参数，不需要添加到参数表
@Parameter(excluded = true)
    public boolean isExported() {
        return exported;
    }
    
```

在appendParameter可以看到对该注解的处理

```
Parameter parameter = method.getAnnotation(Parameter.class);
if (method.getReturnType() == Object.class || parameter != null && parameter.excluded()) {
    continue;
}

```

> ServiceConfig

**1 服务配置对象有哪些重要的属性和机制？**

* 属性继承机制

配置对象存在重复的属性，配置对象之间是相互依赖的关系，如果本身没有配置，可以从其他对象中继承过来

例如，ApplicaitonConfig添加了registry属性，ServiceConig可以从ApplicaitonConfig中继承

* 核心属性

	* interface 接口名
	* ref 接口实现
	* registry 注册中心配置
	* application 应用配置
	* export 决定是否对该服务接口进行暴露
	* delay 是否延迟暴露

* 配置属性检查

服务暴露前会认真检查每个核心配置属性，对于必须的属性，没有则去创建一个并通过刷新(refresh)的方式填充这些属性对象的值

检查的顺序：

```
provider -> application -> registry -> protocol -> metadataReport -> registryDataConfig -> stubAndLocal -> mock
```

* 配置优先级机制

参考 AbstractConfig的refresh方法

服务暴露前（ServiceConfig），会对所有配置属性进行check，提供一种可以从后台(命令行)加载属性的机制（调用refresh方法），即除了可以通过api,xml，注解等方式显示指定属性之外，也可以从

* dubbo属性文件 <https://dubbo.gitbooks.io/dubbo-user-book/configuration/properties.html>
* jvm系统属性
* 环境变量

等后台模式获取，例如-Ddubbo.registry.address

并且，刷新的配置会覆盖原本的配置，即优先级更高

```
SystemConfiguration -> ExternalConfiguration -> AppExternalConfiguration -> AbstractConfig -> PropertiesConfiguration

```

* 泛化调用

服务暴露前检查ref配置属性，判断接口是否为generic

```

 if (ref instanceof GenericService) { //generic service
            interfaceClass = GenericService.class;
            if (StringUtils.isEmpty(generic)) {
                generic = Boolean.TRUE.toString();
            }
        } else { //非generic service
            try {
                //获取接口类
                interfaceClass = Class.forName(interfaceName, true, Thread.currentThread()
                        .getContextClassLoader());
            } catch (ClassNotFoundException e) {
                throw new IllegalStateException(e.getMessage(), e);
            }
            //检查接口，如果没有在应用中找到接口，或接口没有实现，都会报错：
            checkInterfaceAndMethods(interfaceClass, methods);
            checkRef();
            generic = Boolean.FALSE.toString();
        }

```

如果一个接口是generic，则代表改接口支持泛化调用：

参考 <http://dubbo.apache.org/zh-cn/blog/dubbo-generic-invoke.html>

```
泛化调用不需要消费者引入接口模块，可通过接口GenericService调用所有方法

//provider
serviceConfig.setGeneric("true");
serviceConfig.export();

//consumer
referenceConfig.setGeneric(true);
GenericService demoService = (GenericService) referenceConfig.get();
String name = (String) demoService.$invoke("sayHello", new String[]{"".getClass().getName()}, new Object[]{"pony"});


```

---

番外：

如何查找方法提交的历史记录？

**利用idea，选中方法， git -> show history for selection**




