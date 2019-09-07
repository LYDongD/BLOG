## starter机制

### 定义一个springboot-starter，实现依赖的自动配置装配

#### 定义一个springboot starter工程

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.7.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.demo</groupId>
	<artifactId>demo-spring-boot-starter</artifactId>
	<version>0.0.2-SNAPSHOT</version>
	<name>demo-spring-boot-starter</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
	</dependencies>

</project>


```

#### 定义自动配置装配组件

添加@Configuration注解并从属性类中加载属性

```
@Configuration
@EnableConfigurationProperties(value = DemoProperties.class)
//要求属性demo.isopen=true才启用该配置
@ConditionalOnProperty(
        prefix = "demo",
        name = "isopen",
        havingValue = "true"
)
public class DemoConfig {

    @Autowired
    private DemoProperties demoProperties;

    @Bean(name = "demo")
    public DemoService demoService() {
        return new DemoService(demoProperties.getSayWhat(), demoProperties.getToWho());
    }

}


``` 

#### 添加属性类

该类的属性从外部属性文件中加载，例如appLicaiton.properties

通过注解@ConfigurationProperties实现

```
@ConfigurationProperties(prefix = "demo")
public class DemoProperties {

    private String sayWhat;
    private String toWho;


    public String getSayWhat() {
        return sayWhat;
    }

    public void setSayWhat(String sayWhat) {
        this.sayWhat = sayWhat;
    }

    public String getToWho() {
        return toWho;
    }

    public void setToWho(String toWho) {
        this.toWho = toWho;
    }
}

```

#### 【重要】将配置类添加到spring.facoties中

springboot启动后，会进行BEAN资源定位，资源定位方式有2种

* 1 根据@ComponentScan或主类进行包扫描，加载@Component注释的类
* 2 加载META-INF下的spring.factories中定义的接口扩展类

```
//META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.demo.demospringbootstarter.DemoConfig

```

#### 在工程中引入starter依赖

将starter进行打包安装后(注意不需要打成可执行jars,即不需要把它依赖的其他jar打包进来）, 在其他工程中引入

```
        <dependency>
            <groupId>com.demo</groupId>
            <artifactId>demo-spring-boot-starter</artifactId>
            <version>0.0.2-SNAPSHOT</version>
        </dependency>

```

应用启动后，将自动加载starter(spring.factories）指定的配置类，实现配置的自动装配

记得在属性文件中添加自动配置装配组件所需的属性

```
//application.properties
demo.isopen=true
demo.say-what="hello"
demo.to-who="liam"


```
