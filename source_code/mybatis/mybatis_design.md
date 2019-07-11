## mybatis模块涉及思想

### cache缓存

mybatis缓存和其他jvm缓存一样，共享jvm堆内存，因此在设计缓存时，要避免缓存耗尽进程资源

* 缓存应该有过期机制(淘汰策略)
* 缓存应该限制容量
* 缓存仅适合读多写少的场景
* 缓存应该有刷新策略：定时刷新和写时刷新

如果必须要缓存大数据，可以考虑分布式缓存(缓存中间件，如redis,memcached等)

Cache 是装饰器模式的实现范例，通过装饰器组合cache,实现不同策略下的cache


### transaction 事务

mybatis事务可以看成是connection连接的包装，在连接的基础上包了一层事务的控制

例如jdbc的事务实现

```
public class JdbcTransaction implements Transaction {
	protected Connection connection;
	protected DataSource dataSource;
	protected TransactionIsolationLevel level;
	protected boolean autoCommit;

    //事务提交，检查连接属性是否为autoCommit，不是则委托connection进行提交
    @Override
    public void commit() throws SQLException {
      if (connection != null && !connection.getAutoCommit()) {
        if (log.isDebugEnabled()) {
          log.debug("Committing JDBC Connection [" + connection + "]");
        }
        connection.commit();
      }
    }

}


```

如果mybatis和spring容器整合使用，则将事务管理交给spring-jdbc来完成

[参考SpringManagedTransaction](https://my.oschina.net/fifadxj/blog/785621)


