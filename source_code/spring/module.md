#spring 模块分析


### 模块图

![spring架构](https://raw.githubusercontent.com/LYDongD/graphic/master/markdown/spring-structure.png)

### 功能分析

#### 核心模块

* spring-core：Spring中的核心工具类包。
* spring-beans：Spring中定义bean的组件。
* spring-context：Spring的运行容器。
* spring-context-support：Spring容器的扩展支持。
* spring-context-indexer： Spring容器索引。
* spring-expression：Spring的表达式语言支持。

#### 切面

* spring-aop：基于代理的AOP支持。
* spring-aspects：集成Aspects的AOP支持。

#### WEB

* spring-web：提供web的基础功能。
* spring-webmvc：提供springmvc的功能。
* spring-websocket：提供web socket支持。
* spring-messaging：提供web socket STOPM消息支持
* spring-webflux：提供反应式web应用支持

#### 数据访问/集成
* spring-jdbc：提供对jdbc连接的封装功能。
* spring-tx：提供对事务的支持。
* spring-orm：提供对象－关系映射支持。
* spring-oxm：提供对象－XML映射支持。
* spring-jms：提供消息队列的支持。

#### 其他

* spring-expression： 提供强大的spring表达式支持。
* spring-test： 提供测试集成。
* spring-jcl: 提供日志集成。
* spring-instrument：提供instrument支持






