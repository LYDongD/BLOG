## hbase 连接配置

### 使用hbase shell 连接

1 修改hbase_site.xml配置

vim /usr/local/Cellar/hbase/1.2.6/libexec/conf/hbase-site.xml

```
<configuration>
<property>
        <name>hbase.rootdir</name>
        <value>file:///usr/local/var/hbase</value>
  </property>
  <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
  </property>
  <property>
        <name>hbase.zookeeper.quorum</name>
        <value>hadoop-09,hadoop-07,hadoop-08</value>
  </property>
  <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2181</value>
  </property>
</configuration>

```

2 修改/etc/hosts，更改域名映射

```
10.204.58.36 hadoop-07
10.204.58.37 hadoop-08
10.204.58.38 hadoop-09

```

注意事项：

* hbase依赖zk管理元数据，因此可以通过zk连通性，zk上hbase节点是否存在等确认hbase集群是否正常
* 可以访问http://10.204.58.36:60010/master-status，即hbase的web端口，查看hbase的配置和表
* 配置hbase shell 连接时，要考虑域名转换的问题
