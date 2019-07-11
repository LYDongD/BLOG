## springboot 集成kafka

> 1 添加依赖

```
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.1.6.RELEASE</version>
</dependency>

```

> 2 添加配置,从disconf加载

application.yml

```
spring:
  profiles:
    active: @spring.profiles.active@

```

application-local.yml

```
# server config
server:
  port: 8080
spring:
  application:
    name: fcbox-heart-center

#disconf config
disconf:
  scanPackage: com.fcbox.heart.server.disconf
  conf_server_host: 10.204.56.41
  version: "1_0_0_0"
  files: HeartBeatAllConfig.properties
  app: cabinet-heart-server
  env: local
  debug: false
  enable:
    remote:
      conf: true
    system:
      property: true
  conf_server_url_retry_times: 3
  conf_server_url_retry_sleep_seconds: 5
  user_define_download_dir: /app/spring-boot/disconf/cabinet-heart-server
  enable_local_download_dir_in_class_path: false

```

激活local环境

```
<profiles>
        <profile>
            <id>local</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <spring.profiles.active>local</spring.profiles.active>
            </properties>
        </profile>
<profiles>

```

disconf配置, springboot将采用autoConfig模式

```
spring.kafka.consumer.bootstrap-servers=10.204.241.57:9092
spring.kafka.consumer.group-id=fool
spring.kafka.consumer.auto-offset-reset= earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer

```

> 3 消费者

```
@Component
public class KafkaConsumerListener {

    @KafkaListener(topics = "test")
    public void receive(@Payload String message,
                        @Headers MessageHeaders headers) {
        log.info("received message='{}'", message);
        headers.keySet().forEach(key -> log.info("{}: {}", key, headers.get(key)));
    }
}

```

> 4 日志级别配置

```
 <springProfile name="local, dev, uat, uat2, uat3, prd"></springProfile>

 <!-- 开发环境 -->
 <springProfile name="local">
    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>
 </springProfile>

```
