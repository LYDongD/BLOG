## springboot 应用的生命周期

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
