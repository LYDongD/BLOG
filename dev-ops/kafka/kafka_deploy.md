## kafka 单节点部署流程

> 1 下载安装包

```
查看镜像网站最新kafka的安装包并下载：http://mirror.bit.edu.cn/apache/kafka/

例如：

wget http://mirror.bit.edu.cn/apache/kafka/2.2.0/kafka_2.12-2.2.0.tgz

```

> 2 修改kafka的配置文件

```
vim config/server.properties

broker.id=1 //修改默认的broker id
log.dirs=data/kafka-logs //修改持久化存储目录
advertised.listeners=PLAINTEXT://10.204.241.57:9092 //这里需要更改为内网ip地址，内网其他机器无法访问kafka

```

> 3 启动zk

```
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

//通过zkcli验证zk启动是否成功
zkCli -server 10.204.241.57:2181

```

> 4 启动kafka

```
bin/kafka-server-start.sh config/server.properties &

```

> 5 创建topic

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

```

> 6 本地启动producer访问kafka broker

```
kafka-console-producer --broker-list 10.204.241.57:9092 --topic test

```

> 7 本地启动consumer轮询kafka broker

```
kafka-console-consumer --bootstrap-server 10.204.241.57:9092 --topic test

```

> 8 其他命令

```
//远程创建topic
kafka-topics --create --zookeeper 10.204.241.57:2181 --replication-factor 1 --partitions 1 --topic test2

//查看topic的消费组
kafka-consumer-groups --bootstrap-server 10.204.241.57:9092 --list

```

> 常见问题

1 无法连接kafka, 用telnet调试端口，无法访问

* 可能是防火墙的问题，使用systemctl stop firewalld.service 关闭防火墙即可
