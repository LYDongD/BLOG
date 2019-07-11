##dubbo SPI机制

#### dubbo SPI解决的问题

* 延迟实例化
* 自适应(运行时根据不同条件动态生成代码并编译执行)
* 对IOC和aop支持
    
#### 核心类

ExtensionLoader

* 一个扩展点接口对应一个ExtensionLoader
* 核心功能：

**1 扫描dubbo spi 配置文件，将接口的扩展类按类型加载到不同缓存**
    * 自适应扩展类, @Adaptive注解的类 -> cachedAdaptiveClass: Class
    * 包装类, 包含以接口为参数的构造器 -> cachedWrapperClasses: ConcurrentHashSet
    * 激活的扩展类，@Active注解的类 -> cachedActivates: ConcurrentHashMap
    * 普通扩展类(包括被激活的类) -> cachedNames: ConcurrentHashMap, cachedClasses:Holder


**2 延迟实例化扩展类并进行缓存优化**


```java
    @SuppressWarnings("unchecked")
    public T getExtension(String name) {

        //名称不能为空
        if (name == null || name.length() == 0)
            throw new IllegalArgumentException("Extension name == null");

        //如果参数时"true"， 获取默认的扩展实现类
        if ("true".equals(name)) {
            return getDefaultExtension();
        }

        //优先从扩展实例缓存获取
        Holder<Object> holder = cachedInstances.get(name);
        if (holder == null) {
            cachedInstances.putIfAbsent(name, new Holder<Object>());
            holder = cachedInstances.get(name);
        }

        //从holder中获取，涉及容器读写，需要确保线程安全
        Object instance = holder.get();
        if (instance == null) {
            synchronized (holder) {
                instance = holder.get();
                if (instance == null) {
                    //没有则创建扩展实例(这里实现了扩展实例的延迟初始化)
                    instance = createExtension(name);
                    holder.set(instance);
                }
            }
        }
        return (T) instance;
    }


```

**3 扩展类的自动包装**

实例化扩展类时，如果该接口的装饰器器类存在，则实例化装饰器并持有该扩展类

装饰器可添加监听器链或过滤器链，为实际的扩展对象增加切面功能

```

    private T createExtension(String name) {

        //获取该名称的扩展实现类
        Class<?> clazz = getExtensionClasses().get(name);
        if (clazz == null) {
            throw findException(name);
        }

        //优先从缓存获取，没有则实例化
        try {
            T instance = (T) EXTENSION_INSTANCES.get(clazz);
            if (instance == null) {
                EXTENSION_INSTANCES.putIfAbsent(clazz, clazz.newInstance());
                instance = (T) EXTENSION_INSTANCES.get(clazz);
            }

            //注入依赖的属性
            injectExtension(instance);

            //扩展点自动包装： 创建instance的Wrapper装饰器，并注入属性
            Set<Class<?>> wrapperClasses = cachedWrapperClasses;
            if (wrapperClasses != null && !wrapperClasses.isEmpty()) {
                for (Class<?> wrapperClass : wrapperClasses) {
		    //层层包装
                    instance = injectExtension((T) wrapperClass.getConstructor(type).newInstance(instance));
                }
            }
            return instance;
        } catch (Throwable t) {
            throw new IllegalStateException("Extension instance(name: " + name + ", class: " +
                    type + ")  could not be instantiated: " + t.getMessage(), t);
        }
    }

```

**4 扩展点自动依赖注入**

```
   //扩展点依赖注入
    private T injectExtension(T instance) {
        try {
            if (objectFactory != null) {

                //遍历扩展类的所有方法
                for (Method method : instance.getClass().getMethods()) {

                    //处理set方法实现依赖注入(public set(Prameter1))
                    if (method.getName().startsWith("set")
                            && method.getParameterTypes().length == 1
                            && Modifier.isPublic(method.getModifiers())) {

                        //根据publuc set方法的参数找依赖类
                        Class<?> pt = method.getParameterTypes()[0];
                        try {
                            //从method name 获取属性名
                            String property = method.getName().length() > 3 ? method.getName().substring(3, 4).toLowerCase() + method.getName().substring(4) : "";
                            //从对象工厂(IOC)中获取依赖类的对象，根据名称获取
                            Object object = objectFactory.getExtension(pt, property);
                            //获取到则调用set方法实现注入
                            if (object != null) {
                                method.invoke(instance, object);
                            }
                        } catch (Exception e) {
                            logger.error("fail to inject via method " + method.getName()
                                    + " of interface " + type.getName() + ": " + e.getMessage(), e);
                        }
                    }
                }
            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
        return instance;
    }


```

**5 扩展类的自适应**

扩展类的自适应器调用自适应方法时会根据方法的参数（例如协议名）创建适合的扩展对象

如果扩展类是可装饰的，创建扩展类时会利用装饰器模式创建其包装类并实现自动依赖注入

```

   //动态创建自适应类：创建类源码，编译，返回
    private Class<?> createAdaptiveExtensionClass() {
        //生成自适应扩展类的源代码()，核心逻辑：扩展类的名称动态从自适应方法的url参数中获取
        String code = createAdaptiveExtensionClassCode();

        ClassLoader classLoader = findClassLoader();

        // Dubbo SPI 加载 Compier 拓展实现对象对源码进行编译并通过classLoder加载类对象
        org.apache.dubbo.common.compiler.Compiler compiler = ExtensionLoader.getExtensionLoader(org.apache.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();
        return compiler.compile(code, classLoader);
    }


```
