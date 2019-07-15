## springboot dubbo quick start

### 添加依赖

> 注意版本

```
<version.starter.dubbo>0.2.0</version.starter.dubbo>

<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>${version.starter.dubbo}</version>
</dependency>

```

### 添加配置

> 该配置会被springboot auto config 机制加载

```
dubbo.application.id=fcboax-query-center
dubbo.application.name=demo-query-center
dubbo.protocol.id=dubbo
dubbo.protocol.name=dubbo
dubbo.protocol.port=12346
dubbo.registry.address=zookeeper://localhost:2181
dubbo.registry.id=demo-base-center-registry
dubbo.scan.base-packages=com.demo.edms.terminal.facade

```

> 通过disconf引入配置, 在application.yml内添加

```
disconf:
  scanPackage: com.demo.query.server.disconf
  conf_server_host: xx.xx.xx.xx
  version: "1_0_0_0"
  files: multi-db.properties, dubbo.properties
  app: cabinet-query-server
  env: local
  debug: false
  enable:
    remote:
      conf: true //是否拉取远程配置
    system:
      property: true
  conf_server_url_retry_times: 3
  conf_server_url_retry_sleep_seconds: 5
  user_define_download_dir: /app/spring-boot/disconf/cabinet-query-server
  enable_local_download_dir_in_class_path: false

```

### 添加一个dubbo provider service

> 使用注解@com.alibaba.dubbo.config.annotation.Service，注意和spring的@Service区分

```
@com.alibaba.dubbo.config.annotation.Service
public class DemoServiceImpl implements DemoService {

    @Override
    public void hello() {
        System.out.println("hello world");
    }
}

```

### 开启dubbo，监听指定的端口

> 使用@EnableDubbo开启

```
@SpringBootApplication
@EnableDubbo
public class CabinetQueryServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(CabinetQueryServerApplication.class, args);
	}

}

```
