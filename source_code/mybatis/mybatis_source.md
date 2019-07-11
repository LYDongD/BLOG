## mybatis源码阅读心得

### 构造器最佳实践

1 工具类私有化构造器 

如果工具类仅提供静态工具方法，可以将默认构造器声明为private，避免对象被不安全使用

```
public class ExceptionUtil {

   private ExceptionUtil() {
    // Prevent Instantiation
  }
}
```

2 定义静态构造方法提供默认的构造实现

object为空时，使用默认的构造对象

```
public static MetaObject forObject(Object object, ObjectFactory objectFactory, ObjectWrapperFactory objectWrapperFactory, ReflectorFactory reflectorFactory) {
  if (object == null) {
    return SystemMetaObject.NULL_META_OBJECT;
  } else {
    return new MetaObject(object, objectFactory, objectWrapperFactory, reflectorFactory);
  }
}
```

### 空对象处理机制

1 安全获取属性，提供默认值

如果属性为空，则使用默认值，该值由外部传入

```
private String getPropertyValue(String key, String defaultValue) {
  return (variables == null) ? defaultValue : variables.getProperty(key, defaultValue);
}

```

2 使用Optional简化空判断处理

```
String argTypes = Optional.ofNullable(constructorArgTypes).orElseGet(Collections::emptyList)
    .stream().map(Class::getSimpleName).collect(Collectors.joining(","));

```

3 集合对空key的处理

使用computeIfAbsent，传入一个func，为null key 生成一个key值

```
//范例1，简化空key处理
List<Method> list = conflictingMethods.computeIfAbsent(name, k -> new ArrayList<>());

//范例2 一行代码实现缓存读写
public Reflector findForClass(Class<?> type) {
  if (classCacheEnabled) {
    // synchronized (type) removed see issue #461
    return reflectorMap.computeIfAbsent(type, Reflector::new);
  } else {
    return new Reflector(type);
  }

```

### 反射

1 通过Constructor创建对象

可通过参数类型列表，绑定到指定构造方法，相比之下，Class.newInstance只能使用default 构造器创建对象

```
class.getDeclaredConstructor(Class<?>... parameterTypes).newInstance()

```

可以修改Constructor的访问权限，从而可以通过private或protected构造方法实例化

```
 Constructor<CellFaultService> constructor = CellFaultService.class.getDeclaredConstructor();
 constructor.setAccessible(true);
 FaultService cellFaultService = constructor.newInstance();
 Dispatcher.putService(cellFaultService, FaultTypeEnum.CELL_NOT_OPEN, FaultTypeEnum.CELL_NOT_CLOSE);

```

### 语法糖

1 冒号操作符

* 实际上是调用类的特定方法，是一种lambda下func call 的一种简写

模式：类::方法(不带括号)

```

person -> person.getAge(); => Person::getAge
x -> System.out.println(x); => System.out::println
k -> new Reflection(); => Reflector::new

```



### 多线程实践

1 连接池获取连接线程同步机制

基于对象锁：PoolState，实现线程等待通知的同步机制

获取连接时，无空闲连接，且当前活跃连接已达上限，则需要等待

```

while (conn == null) {
	….
	synchronized (state) {
		….
		try {
			if state.idleConnections.isEmpty() && state.activeConnections.size() >= poolMaximumActiveConnections && longestCheckoutTime <= poolMaximumCheckoutTime {
				…
				state.wait(poolTimeToWait); 
				…
			}
		}catch(InterruptedException e) {
			break;
		}
		

		…
	}
}

```

连接归还池后，通知等待队列，唤醒等待线程

```
synchronized (state) {
	…
	state.notifyAll();
	…
}

```

### 枚举类使用范式

优化根据code获取枚举实例的方法

使用静态字典保存枚举状态码和枚举实例之间的映射，避免获取实例时进行全枚举扫描

```

private static Map<Integer,JdbcType> codeLookup = new HashMap<>();

static {
  for (JdbcType type : JdbcType.values()) {
    codeLookup.put(type.TYPE_CODE, type);
  }
}

public static JdbcType forCode(int code)  {
  return codeLookup.get(code);
}

```

### 弱引用缓存

WeakCache，缓存的value是一个弱引用

* 定义WeakEntry扩展WeakReference，添加key属性，key是强引用
* WeakEntry关联ReferenceQueue， value对象被GC时，WeakEntry添加到队列，执行特定操作时，poll队列删除无效key

```
private static class WeakEntry extends WeakReference<Object> {
  private final Object key;

  private WeakEntry(Object key, Object value, ReferenceQueue<Object> garbageCollectionQueue) {
    super(value, garbageCollectionQueue);
    this.key = key;
  }
}


//删除无效key
private void removeGarbageCollectedItems() {
  WeakEntry sv;
  while ((sv = (WeakEntry) queueOfGarbageCollectedEntries.poll()) != null) {
    delegate.removeObject(sv.key);
  }
}

```

**弱引用的其他应用场景：**

定义ThreadLocal变量，线程通过一个字典持有本地线程key对象及其value，其中，对ThreadLocal key 对象的引用是弱引用

* 如果没有其他强引用时，线程不会阻碍key被GC
* 被置空，导致线程本地字典，hashmap持有一个null key
* get/put操作线程本地变量时，会清理一次null key，但是如果长时间未调用该方法会导致内存泄露
* 使用完ThreadLocal后，显示调用remove方法清理null key指向的value避免内存泄露


其他引用定义和使用参考：

[4种引用的定义和区别](https://juejin.im/post/5a5129f5f265da3e317dfc08)


### 链式语法

链式语法通常用在构建起(Builder)之上，增强了构建的可读性

构建缓存：

```
Cache cache = new CacheBuilder(currentNamespace)
    .implementation(valueOrDefault(typeClass, PerpetualCache.class))
    .addDecorator(valueOrDefault(evictionClass, LruCache.class))
    .clearInterval(flushInterval)
    .size(size)
    .readWrite(readWrite)
    .blocking(blocking)
    .properties(props)
    .build();

```

构造属性方法将返回自身的引用：

```
public CacheBuilder addDecorator(Class<? extends Cache> decorator) {
  if (decorator != null) {
    this.decorators.add(decorator);
  }
  return this;
}

```


