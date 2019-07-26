## springboot 应用的生命周期

### 如何启动一个standalone的springboot应用

springboot应用有三种类型： NONE/SERVLET/REACTIVE， 其中NONE不需要启动一个web容器

```
@SpringBootApplication
@ComponentScan(basePackages = "com.fcbox.connect")
@Slf4j
public class CabinetPushServerApplication implements CommandLineRunner {

	public static void main(String[] args) {
		SpringApplication.run(CabinetPushServerApplication.class, args);
	}

    @Override
    public void run(String... args) throws Exception {
        Thread.currentThread().join();
    }
}

```

### 如何在应用启动完毕后执行相关业务？

类似于ApplicationListener, 使用ApplicationRunner实现

```
//注意，必须加上@Component注解，否则该类不会在启动时被加载
@Component
public class TestImplApplicationRunner implements ApplicationRunner {
 
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(args);
        System.out.println("这个是测试ApplicationRunner接口");
    }
}

```
