## transient 关键字

### 用法

* 修饰成员变量
* 该成员变量不会被序列化，即使类是可序列化的

### 举例


** ArrayList中存储元素的数组**

```
//缓存数组
transient Object[] elementData;

```

该数组为缓存数组，会预留一部分容量，容量不足会扩充容量，可能有部分空间未存储实际元素,
为了仅序列化实际存储的元素，采用了禁止序列化缓存数组的方式，转为下面的方式：

```

//分别对size和每个元素进行序列化，确保持久化的元素是实际存在的
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{

    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();
 
    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);
 
    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }
 
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}


```

**dubbo中ServiceConfig中大量使用了transient修饰成员变量,这些字段不会进行io传输**

```
    private static transient ApplicationContext SPRING_CONTEXT;

    private final transient Service service;

    private transient ApplicationContext applicationContext;

    private transient String beanName;

    //transient修饰的boolean
    private transient boolean supportedApplicationListener;

```


**对于密码等安全级别较高的对象，通常声明为transient**


