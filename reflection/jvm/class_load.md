## 类加载机制

### 流程

* 加载
   * 双亲委托模式下的类加载器实现类加载（至上而下委托）
		* boot class loader
		* java.lang ClassLoader
			* extension class loader
			* application class loader
			* 自定义ClassLoder
    * 本质上是将字节码文件（字节流）加载到内存方法区(perm area)，并构建类对象
    * 可通过参数 -XXPermSize/XXMaxPermSize进行设置方法区大小
* 链接
    * 验证
		* 根据jvm约束进行验证
    * 准备
	   * 为静态字段分配内存，为初始化做准备
	   * 构造相关数据结构，例如用于动态绑定的方法表
    * 解析：将符号引用解析为实际引用
	   * 静态绑定：方法符号被解析成目标方法的引用
	   * 动态绑定：方法符号被解析成类方法表的索引
* 初始化(类仅会初始化一次)
    * 常量/static block
    * 类只有初始化完成后才能使用
	


### 查看类加载

**使用类初始化机制实现单例**

```

/**
 * 使用内部类初始化单例，利用类只会初始化一次的机制保证单例的唯一性
 */
public class MySingleton {
    
    //禁止默认构造方法，避免单例被随意创建
    private MySingleton() {
    }

    //持有静态内部类，首次获取单例时初始化该内部类创建单例
    private static class SingletonHolder {

        //类初始化时会初始化静态final成员变量,相当于初始化常量,常量一般大写
        static final MySingleton INSTANCE = new MySingleton();

        //测试类初始化
        static {
            System.out.println("SingletonHolder <cinit>");
        }
    }

    public static Object getSingletonn(boolean initArray) {
        //返回数组或单例, new引用类型的数组不会初始化
        if(initArray) return new SingletonHolder[2];
	//访问static final 成员变量(常量)时会被初始化
        return SingletonHolder.INSTANCE;
    }

    public static void main(String args[]) {
        MySingleton.getSingletonn(true);
        System.out.println("---------");
        MySingleton.getSingletonn(false);
    }

```

**使用java -XX:+TraceClassLoading 打印类加载日志**

```

...

[Loaded java.lang.Class$MethodArray from /Library/Java/JavaVirtualMachines/jdk1.8.0_73.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Void from /Library/Java/JavaVirtualMachines/jdk1.8.0_73.jdk/Contents/Home/jre/lib/rt.jar]
//构造引用类型数组会触发类的加载但是不会初始化
[Loaded com.liam.demo.singleton.MySingleton$SingletonHolder from file:/Users/lee/liam-demo/src/]
---------
//获取static final 成员变量会触发类的初始化
SingletonHolder <cinit>
[Loaded java.lang.Shutdown from /Library/Java/JavaVirtualMachines/jdk1.8.0_73.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Shutdown$Lock from /Library/Java/JavaVirtualMachines/jdk1.8.0_73.jdk/Contents/Home/jre/lib/rt.jar]

...

```

* jre下lib/的jar包一般是由boot class loader完成加载
* jre下lib/ext/的jar包一般是由extention class loder完成加载
* classpath(-cp/-classpath)下或系统环境变量CLASSPATH下的jar一般是由application class loder负责加载




