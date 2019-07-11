## 阿里巴巴规约解读

### 序列化

序列化是将对象的状态信息转换为可存储或传输的形式的过程

#### jvm的序列化与反序列化

* 通过ObjectInputStream和ObjectOutputStream实现
* 反序列化时会校验版本：serialVersionUID, 不一致会抛出异常
    * 兼容性升级，例如增加或减少一些字段，不必改变该字段
    * 非兼容性升级，必须修改该字段
    * 该字段可通过IDEA配置自动生成

#### 日志

* 日志
    * 使用SLF4J, 门面模式的典范

#### POJO

* pojo中的布尔类型
    * 避免使用isXX的形式，对于利用反射遍历getter方法获取属性的序列化框架来说，可能找不到属性或反序列化失败
* 可序列化pojo总的serialVersionUID
    * 添加并谨慎修改，避免jvm反序列化失败


#### String

* 字符串拼接的最佳方式
    * 单线程：StringBuilder
    * 多线程:  StringBuffer
    * 避免在循环体内使用+，带来频繁创建StringBuilder的开销

#### 集合

* 如何过滤集合
    * failfast机制在List中生效的原因
    * 通过迭代器移除元素
* 集合初始化，容量定义公式
    * capacity / factor + 1
    * hashmap容量初始化原理(位操作) -> 将容量设置为大于当前传入容量的最小2^k数
        * 例如传入7，则设置为8；传入13，则设置为16
