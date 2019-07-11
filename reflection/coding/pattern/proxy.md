## 代理模式

### 定义

代理目标对象并取得对象的控制权

### 角色

* 目标对象： target
* 代理对象： targetProxy
* 代理拦截器：targetIntecepor

代理拦截器取得目标对象的控制权，并自定义切面逻辑


### 分类

* 静态代理
	* 需要为每一个目标类创建一个唯一的代理类

* 动态代理
	* 仅需要需要实现代理拦截器，运行时动态为目标对象创建代理

### 动态代理实现

#### 方式1: 基于接口的动态代理(jdk)

代理拦截器需要实现InvocationHandler取得目标对象的控制权


```

package com.liam.demo.proxy.jdk;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;


/**
 * The {@code LoggerInterceptor}
 *
 * 基于jdk实现的动态代理拦截器，可代理接口实现类
 * 拦截器具有被代理对象的控制权
 *
 * @author  liam
 * @version 1.0
 */
public class JdkLoggerInterceptor implements InvocationHandler {


    //被代理对象(目标对象)
    private Object target;

    public JdkLoggerInterceptor(Object target) {
        super();
        this.target = target;
    }

    public JdkLoggerInterceptor(){};


    //直接创建代理对象
    public Object createProxy(Object targetObject){

        this.target = targetObject;

        Class<?> targetClass = targetObject.getClass();
        return Proxy.newProxyInstance(targetClass.getClassLoader(), targetClass.getInterfaces(), this);

    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        System.out.println("before saying hello");

        //需要传入被代理对象和参数
        method.invoke(target, args);

        System.out.println("after saying hello");
        return null;
    }
}


```

#### jdk代理的原理

jdk代理实际上是利用java的反射机制，创建代理对象的时候获取所有接口的方法，并在调用目标方法时通过代理拦截器去调用

```

    //获取代理接口的类对象
    Class<ISpeech>  interfaces = (Class<ISpeech>) Class.forName("com.pubutech.example.patterns.proxy.common.ISpeech");
    
    //获取接口的方法列表
    Method[] methods = interfaces.getMethods();
    
    //取得目标对象方法的控制权
    for (Method i :methods) {
        //使用拦截器进行调用
        interceptor.invoke(speecher);
    }

```

#### 方式2 基于类的动态代理(cglib)

代理拦截器需要实现MethodInterceptor获取目标对象的控制权

```
package com.liam.demo.proxy.cglib;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * The {@code CglibLoggerInterceptor}
 * 基于类的动态代理，目标对象无需实现接口
 * 拦截器具有被代理对象的控制权
 *
 * @author  liam
 * @version 1.0
 */
public class CglibLoggerInterceptor implements MethodInterceptor{

    private Object target;


    public Object createProxy(Object targetObject){
        this.target = targetObject;

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(targetObject.getClass());
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {

        System.out.println("before saying hello");

        //使用methodProxy调用
        Object o1 = methodProxy.invoke(target, objects);

        System.out.println("after saying hello");

        return o1;
    }
}

```

**测试**

```

public static void main(String args[]) {

        //jdk方式1
        //为目标对象创建动态代理，并获得目标对象的控制权
        EnglishGuy englishGuy = new EnglishGuy();
        JdkLoggerInterceptor jdkLoggerInterceptor = new JdkLoggerInterceptor(englishGuy);
        ClassLoader classLoader = englishGuy.getClass().getClassLoader();
        Person personProxy = (Person)Proxy.newProxyInstance(classLoader, englishGuy.getClass().getInterfaces(), jdkLoggerInterceptor);
        //目标对象的调用实际上通过动态代理调用，动态代理通过拦截器(InvocationHandler)实现调用
        personProxy.sayHello();
        System.out.println("--------");

        //2 jdk方式2，进一步封装
        Person personProxy1 = (Person)new JdkLoggerInterceptor().createProxy(englishGuy);
        personProxy1.sayHello();
        System.out.println("--------");

        //cglib方式
        EnglishGuy englishGuyProxy = (EnglishGuy) new CglibLoggerInterceptor().createProxy(englishGuy);
        englishGuyProxy.sayHello();

}
    
    
//测试结果
    
before saying hello
hello world
after saying hello
--------
before saying hello
hello world
after saying hello
--------
before saying hello
hello world
after saying hello

```




#### 总结

* 代理模式中动态代理常用于动态增加功能，例如spring aop的实现
* jdk动态代理基于接口实现，目标对象必须实现接口
* cglib通过扩展目标对象子类实现，目标对象不能是final类型


