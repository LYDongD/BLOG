## spring xml 扩展


### xml配置文件的namespace机制

xmlns -> xml schema -> xml schema definition

* xml schema 定义了xml的文档结构
* xml schema definition(xsd) 定义了xml schemea
* xsd定义了：
	* 什么元素是合法的
	* 子元素的数量和顺序
	* 元素及其属性类型
	* 默认值

---

### spring schema 扩展

Spring提供了可扩展Schema的支持，这是一个不错的折中方案，完成一个自定义配置一般需要以下步骤： 

```
1、设计配置属性和JavaBean 
2、编写XSD文件 
3、编写NamespaceHandler和BeanDefinitionParser完成解析工作 
4、编写spring.handlers和spring.schemas串联起所有部件 
5、在Bean文件中应用
```

----

#### 需求：自定义标签配置spring的容器bean

* 它需要被解析成BeanDefinition
* 它自定义一套自己的标签体系
* 它能够被解析成特定的javaBean

### 解决方案

利用spring的xml扩展机制

* 两个关键文件
    * spring.handlers : 定义handler, 用于加载和解析标签
    * spring.schemas: 定义xsd, 即自定义标签的规范和约束

* spring如何加载xml标签？

```
1. spring通过NamespaceHandler加载配置文件
2. NamespaceHandler定义在spring.handlers文件
3. 优先采用默认handler加载，遇到不认识的标签，选择其他自定义handler进行加载

```
* spring 如何约束xml标签？

```
1. 通过xxx.xsd定义标签约束
2. xxx.xsd声明在spring.schemas文件

```

spring启动时会加载：

1. spring.handlers
2. spring.schemas


* 开源组件如何扩展spring的xml配置？

以dubbo为例：

1. 定义元素标签：dubbo.xsd，声明在spring.schemas
2. 定义标签解析器：DubboNamespaceHandler， 声明在spring.handlers
3. 添加依赖时，同时会把dubbo.xsd, spring.schemas, spring.handlers依赖进来，应用启动时会加载该这些文件
    * 如果项目打包没有包含这些文件，将导致xml解析失败

```

//spring.schemas
http\://dubbo.apache.org/schema/dubbo/dubbo.xsd=META-INF/dubbo.xsd
http\://code.alibabatech.com/schema/dubbo/dubbo.xsd=META-INF/compat/dubbo.xsd

//spring.handlers
http\://dubbo.apache.org/schema/dubbo=org.apache.dubbo.config.spring.schema.DubboNamespaceHandler
http\://code.alibabatech.com/schema/dubbo=org.apache.dubbo.config.spring.schema.DubboNamespaceHandler

```

* 在配置文件中添加自定义标签时，需要声明xsd

```
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <dubbo:annotation package="com.clife.bigdata.business.scene" />
	<dubbo:application name="clife-bigdata-business-scene" />
	<dubbo:registry protocol="zookeeper" address="${zookeeper.address}" group="${dubbo.group}"/>
  	<dubbo:protocol name="dubbo" port="${expert.service.dubbo.port}" />

</beans>

```


