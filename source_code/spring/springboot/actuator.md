## Actuator 应用监控

### 用法

[参考](http://www.ityouknow.com/springboot/2018/02/06/spring-boot-actuator.html)

> 添加依赖

通过web容器暴露http端口

```
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
</dependencies>

```

> 配置

properties配置

```
info.app.name=spring-boot-actuator
info.app.version= 1.0.0
info.app.test=test

management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
#management.endpoints.web.base-path=/monitor

management.endpoint.shutdown.enabled=true

```
yml配置

```
info:
   app:
      name: cabinet-query-server
      version: 1.0.0
management:
  health:
    mail:
      enabled: false
  endpoints:
    web:
      exposure:
        # 生产环境由于安全性的问题，注意不要暴露敏感端点
        include: "*"
        exclude: env
      base-path: /actuator
  endpoint:
    health:
      show-details: always

```

> 常用功能

* heap dump
* thread dump
* 健康检查等



