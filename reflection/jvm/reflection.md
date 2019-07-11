## 反射原理

### 什么是反射？

反射是一种java运行时机制，使动态观察和修改程序的行为

**运行时机制的运用**
* 多态 -> 方法的动态绑定 -> 调用方法时才找到目标方法
* 反射 -> 动态获取类对象 -> 进一步观察和修改类行文等
* 字节码增强 -> 动态创建字节码 -> 创建动态类对象

### 反射API

反射API的入口是类对象：

* Class.forName
* 对象的getClass()
* 类.class

在此基础上进行其他反射api扩展：

* 创建类的实例: newInstance()
* 获取类成员: getFields()/getConstructors()/getMethods()
* 方法调用/其他行为: Method.invoke()/Field.get/set()...

### 反射的应用

* 动态代理，获取委托对象方法的控制权
* 配置扩展，例如spring ioc中根据beanDefinition装配bean

### 反射方法的调用原理

* 委派机制
    * 委托本地实现：将调用本地方法
    * 委托动态实现：将动态创建委托类

* inflation机制
    * 根据阙值切换委派实现方式

inflation机制应用还可参考HashMap底层数据结构的切换，当链表节点数达到阙值，将切换成红黑树

```

/**
 * 观察反射方法调用的原理
 *
 * 1 委派机制
 * 2 本地方法调用
 * 3 字节码增强
 * 4 inflation切换
 */
public class MethodInvoker {


    /**
     * 通过反射调用该方法
     *
     * @param i 参数
     */
    public static void target(int i) throws Exception{
        //通过构造异常来追踪方法调用栈信息
        new Exception("#" + i).printStackTrace();
    }

    public static void main(String args[]) throws Exception{

        Class<?> methodInvokerClass = Class.forName("com.liam.demo.reflection.MethodInvoker");
        //参数是可变参数，获取方法时本质是通过数组传递
        Method targetMethod = methodInvokerClass.getMethod("target", int.class);

        //反复调用，委派模式inflation切换：前15次采用本地实现，后5次采用动态实现
        //通过-Dsun.reflect.inflationThreshol=15设定， -Dsun.reflect.noInflation=true 关闭本地实现
        for (int i = 0; i < 20; i++){
            //只有静态方法才可以传null，否则应该传入具体事例
            targetMethod.invoke(null, 1);
        }

    }
}


```

执行结果:

```
//前15次本地调用：

java.lang.Exception: #1
	at com.liam.demo.reflection.MethodInvoker.target(MethodInvoker.java:18)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.liam.demo.reflection.MethodInvoker.main(MethodInvoker.java:31)
java.lang.Exception: #1
	at com.liam.demo.reflection.MethodInvoker.target(MethodInvoker.java:18)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.liam.demo.reflection.MethodInvoker.main(MethodInvoker.java:31)

//后5次动态委托类调用:

java.lang.Exception: #1
	at com.liam.demo.reflection.MethodInvoker.target(MethodInvoker.java:18)
	at sun.reflect.GeneratedMethodAccessor1.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.liam.demo.reflection.MethodInvoker.main(MethodInvoker.java:31)


```

* -Dsun.reflect.inflationThreshol=15 设置inflation阙值
* -Dsun.reflect.noInflation=true 关闭本地调用


### 反射性能

* 反射性能损耗来源：
    * 本地方法调用：Class.forName()
    * 查找方法On的遍历：Class.getMethod()

**解决方案：** 
* 使用HashMap缓存，将时间复杂度降到01
* 方法调用时，关闭本地实现模式，使用动态委托模式


