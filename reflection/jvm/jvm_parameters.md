## jvm 启动参数调优与监控

### 以下是一个典型的jvm启动命令

```
/app/jdk1.8.0_65/bin/java -server -Xms2048m -Xmx2048m -Xmn1g -Xss256K -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/app/spring-boot/gc_logs/CABINET_HEART_SERVER_01/java_dump110886.hprof -Xloggc:/app/spring-boot/gc_logs/CABINET_HEART_SERVER_01/gc110886.log -XX:+PrintGCDetails -XX:+PrintHeapAtGC -XX:+PrintGCDateStamps -XX:+DisableExplicitGC -XX:+ExplicitGCInvokesConcurrent -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=3 -XX:GCLogFileSize=5M -Dsun.rmi.dgc.client.gcInterval=3600000 -Dsun.rmi.dgc.server.gcInterval=3600000 -Dcom.sun.management.jmxremote -Djava.rmi.server.hostname=10.204.51.67 -Dcom.sun.management.jmxremote.port=1050 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -jar /app/war/CABINET_HEART_SERVER_01/pkg/cabinet-heart.jar --server.address=10.204.51.67 --server.port=8080

```

#### 参数分析：

> 内存分配

* -Xms 初始堆，单位支持k,m,g
* -Xmx 最大堆，单位支持k,m,g，与-Xms的值通常被设置为相同
* -Xmn 堆年轻代大小， 用于为新对象分配空间，将影响YGC的频率，建议设置为-Xmx的一半或1/4
* -Xss  栈大小, 64位平台默认1024k
* -XX:MetaspaceSize  元数据初始大小，类二进制存放于此
* -XX:MaxMetaspaceSize 元数据最大值

> GC

* -XX:+UseConcMarkSweepGC 老年代使用CMS gc
* -XX:+UseParNewGC 新生代使用parallel new gc (ParNew + CMS 是固定组合，吞吐量较高，但是可能会产生内存碎片)
* -XX:CMSInitiatingOccupancyFraction  老年代CMS占用整个gc收集周期的比例
* -XX:+UseCMSInitiatingOccupancyOnly 运行占用值作为CMS的唯一标，搭配CMSInitiatingOccupancyFraction使用
* -XX:+DisableExplicitGC 禁用System.gc()调用
* -XX:+ExplicitGCInvokesConcurrent 允许System.gc() 调用并发GC
* -Dsun.rmi.dgc.client.gcInterval  RMI的定时GC触发机制，DGC的触发频率
* -Dsun.rmi.dgc.server.gcInterval RMI的定时GC触发机制，DGC的触发频率


解读：

1 -XX:+DisableExplicitGC 禁用了System.gc()调用full gc, 就没必要设置XX:+ExplicitGCInvokesConcurrent (用并发gc代替fullgc), 两者只需要配置一个

2 没有禁用System.gc()时，jvm默认每一个小时调用System.gc() 进行full gc，可通过-Dsun.rmi.dgc.client.gcInterval和-Dsun.rmi.dgc.server.gcInterval 调整:

[参考](http://hongjiang.info/tomcat-full-gc-every-hour/)


> JMX

```
-Dcom.sun.management.jmxremote 开启jMX
-Djava.rmi.server.hostname RMI host
-Dcom.sun.management.jmxremote.port   RMI 端口
-Dcom.sun.management.jmxremote.ssl 是否开启ssl
-Dcom.sun.management.jmxremote.authenticate 是否启用鉴权

```

解读：

开启JMX后，可远程访问jvm并实现监控或手动gc等

1 使用jconsole， 例如jconsole 10.204.51.67:1050

2 使用jvisualvm

通过可视化方式观察jvm动态消耗资源的情况，例如内存变化等


> 堆/gc 日志转储


```
-XX:+HeapDumpOnOutOfMemoryError 发生OOM时堆转储
-XX:HeapDumpPath 堆转储文件的路径
-XX:+PrintGCDetails 允许打印GC详细信息
-XX:+PrintHeapAtGC  每次GC后都打印堆信息
-XX:+PrintGCDateStamps 运行GC上打印时间戳
-XX:+UseGCLogfileRotation 开启gc日志文件滚动循环
-XX:GCLogFileSize  设置gc日志文件滚动策略：按指定大小
-XX:NumberOfGCLogFiles 设置滚动日志文件的个数
-Xloggc 记录gc事件的日志路径

```


