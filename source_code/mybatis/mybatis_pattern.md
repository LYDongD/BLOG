## mybatis 源码分析之设计模式

### 1 抽象工厂模式

* 抽象工厂接口
* 可实现不同的工厂类
* 不同工厂类可定义不同的生产模式，例如生产不同的抽象对象实现类等

参考[抽象工厂模式](https://github.com/iluwatar/java-design-patterns/tree/master/abstract-factory)

```

public interface ObjectFactory {
    void setProperties(Properties properties);
    <T> T create(Class<T> type);
    <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs);
}

实现类：

DefaultObjectFactory
ExampleObjectFactory
CustomObjectFactory

```

### 2 装饰器模式

* 对目标对象进行装饰，添加装饰逻辑
* 实现目标接口，并注入接口对象
* 代理目标对象的方法，增加装饰逻辑

```
private static class DriverProxy implements Driver {
	private Driver driver;
	
	DriverProxy(Driver d) {
	  this.driver = d;
	}
	
	…..
}

```

cache的实现，使用装饰器组合cache实现不同功能的cache

例如FIFOCache和LRUCache, 分别使用不同的数据结构实现缓存，对应不同的缓存过期策略

```
public class FifoCache implements Cache {
    private final Cache delegate;
    private final Deque<Object> keyList;
    private int size;  //cache capacity

    //缓存失效策略
    private void cycleKeyList(Object key) {
        keyList.addLast(key); //add before delete
        if (keyList.size() > size) {
            Object oldestKey = keyList.removeFirst();
            delegate.removeObject(oldestKey);
        }
    }
}

```

```
public class LruCache implements Cache {
     private final Cache delegate;
     private Map<Object, Object> keyMap;
     private Object eldestKey;

     private void cycleKeyList(Object key) {
         keyMap.put(key, key);
         if (eldestKey != null) {
            delegate.removeObject(eldestKey);
            eldestKey = null;
         }
    }
}

```

实现序列化存储的cache

```

public void putObject(Object key, Object object) {
    if (object == null || object instanceof Serializable) {
      delegate.putObject(key, serialize((Serializable) object));
    } else {
      throw new CacheException("SharedCache failed to make a copy of a non-serializable object: " + object);
    }
}

//采用java序列化协议实现value对象的序列化
private byte[] serialize(Serializable value) {
  try (ByteArrayOutputStream bos = new ByteArrayOutputStream();
       ObjectOutputStream oos = new ObjectOutputStream(bos)) {
    oos.writeObject(value);
    oos.flush();
    return bos.toByteArray();
  } catch (Exception e) {
    throw new CacheException("Error serializing object.  Cause: " + e, e);
  }
}
```

### 3 模板方法

BaseTypeHandler抽象基础类实现了模板方法：

* 异常统一处理
* 参数绑定时，对空参数的统一处理
* 非空参数绑定和结果返回处理交给子类去实现

```

public abstract class BaseTypeHandler<T> extends TypeReference<T> implements TypeHandler<T> {
    @Override
  public void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
    //统一处理空参数
    if (parameter == null) {
      if (jdbcType == null) {
        throw new TypeException("JDBC requires that the JdbcType must be specified for all nullable parameters.");
      }
      try {
        ps.setNull(i, jdbcType.TYPE_CODE);
      } catch (SQLException e) {
        throw new TypeException("Error setting null for parameter #" + i + " with JdbcType " + jdbcType + " . "
              + "Try setting a different JdbcType for this parameter or a different jdbcTypeForNull configuration property. "
              + "Cause: " + e, e);
      }
    } else {
      try {
        setNonNullParameter(ps, i, parameter, jdbcType);
      } catch (Exception e) { //统一处理异常
        throw new TypeException("Error setting non null for parameter #" + i + " with JdbcType " + jdbcType + " . "
              + "Try setting a different JdbcType for this parameter or a different configuration property. "
              + "Cause: " + e, e);
      }
    }
  }
}

//交给子类实现
public abstract void setNonNullParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;

```

### 4 单例模式

IO模块下的VFS类，使用内部静态类实现了一个线程安全的单例

```
public abstract class 	 {
	private static class VFSHolder { 
		static final VFS INSTANCE = createVFS();
		public static VFS getInstance() {
 		 	return VFSHolder.INSTANCE;
		}
	}
}

```

### 5 注册表模式

注册表本质上是一个缓存对象，包含单个或多个集合，对进程内的数据，尤其是元信息进行存储和管理

分布式注册表即注册中心，例如zookeeper, rocketmq的namesrv等

TypeHandlerRegistry

```
public final class TypeHandlerRegistry {

//集合
  private final Map<Type, Map<JdbcType, TypeHandler<?>>> typeHandlerMap = new ConcurrentHashMap<>();

 //注册bean
 public <T> void register(Class<T> javaType, TypeHandler<? extends T> typeHandler) {
 	...
 }
 
 //获取bean
 public TypeHandler<?> getTypeHandler(JdbcType jdbcType) {
 	...
 }
  
}

```
### 6 动态代理

通过动态代理，对目标对象实现切面增强，例如给jdbc相关的组件增强日志功能等

例如ResultSet的动态代理，增加了结果集的打印

通过jdk基于接口实现动态代理：

```
public static ResultSet newInstance(ResultSet rs, Log statementLog, int queryStack) {
  InvocationHandler handler = new ResultSetLogger(rs, statementLog, queryStack);//切面拦截器注入目标对象
  ClassLoader cl = ResultSet.class.getClassLoader();
  return (ResultSet) Proxy.newProxyInstance(cl, new Class[]{ResultSet.class}, handler); //基于接口代理，并注入拦截器
}

//ResultSet的切面拦截器：ResultSetLogger, 实现InvocationHandler
public final class ResultSetLogger extends BaseJdbcLogger implements InvocationHandler {
  
   //该方法将被动态代理反射调用，传入代理，方法和方法参数
  @Override
  public Object invoke(Object proxy, Method method, Object[] params) throws Throwable {
    try {
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, params);
      }
      Object o = method.invoke(rs, params); //委托目标对象调用目标方法，前后注入增强逻辑
      if ("next".equals(method.getName())) {
        if ((Boolean) o) {
          rows++;
          if (isTraceEnabled()) {
            ResultSetMetaData rsmd = rs.getMetaData();
            final int columnCount = rsmd.getColumnCount();
            if (first) { //print column names when iterate first row
              first = false;
              printColumnHeaders(rsmd, columnCount);
            }
            printColumnValues(columnCount); //print column values
          }
        } else {
          debug("     Total: " + rows, false);
        }
      }
      clearColumnInfo();
      return o;
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
  }
}

```

* 动态代理是通过jdk或cglib等库，运行时通过反射创建的代理类，不需要定义一个静态的代理类
* 动态代理通过切面拦截器实现增强逻辑
    * 构造动态代理对象时需要注入切面拦截器，动态代理委托拦截器实现方法调用
    * 切面拦截器通常要持有目标对象，显示的委托目标对象调用目标方法，并在前后注入增强逻辑


cglib基于子类创建动态代理：

```
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(targetObject.getClass()); //设置父类
enhancer.setCallback(handler); //设置拦截器
return enhancer.create();

```

spring aop的实现方式

* 默认
    * Bean实现了接口，则通过jdk动态代理API创建代理对象
    * 否则，通过cglib api 创建动态代理对象

* AOP配置属性：proxy-target-class=true，则强制使用CGLIB创建动态代理

```
<aop:aspectj-autoproxy proxy-target-class="true"/>

<aop:config proxy-target-class="true">
    <aop:aspect id="xxx" ref="xxxx">
        <!-- 省略 -->
    </aop:aspect>
</aop:config>

```

[aop动态代理创建源码分析](https://www.tianxiaobo.com/2018/06/20/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%88%9B%E5%BB%BA%E4%BB%A3%E7%90%86%E5%AF%B9%E8%B1%A1/)

