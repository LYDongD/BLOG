## kafka 相关操作

> 1 查看主题列表及详情

```

kafka-topics --zookeeper 10.204.52.173:2181 --list
kafka-topics --zookeeper 10.204.52.173:2181 --describe --topic cabinetBaseInfoQueue

```

查看指定topic的详情如下

```
	Topic: cabinetBaseInfoQueue	Partition: 0	Leader: 174	Replicas: 174	Isr: 174
	Topic: cabinetBaseInfoQueue	Partition: 1	Leader: 175	Replicas: 175	Isr: 175
	Topic: cabinetBaseInfoQueue	Partition: 2	Leader: 173	Replicas: 173	Isr: 173
	Topic: cabinetBaseInfoQueue	Partition: 3	Leader: 174	Replicas: 174	Isr: 174
	Topic: cabinetBaseInfoQueue	Partition: 4	Leader: 175	Replicas: 175	Isr: 175
	Topic: cabinetBaseInfoQueue	Partition: 5	Leader: 173	Replicas: 173	Isr: 173
	Topic: cabinetBaseInfoQueue	Partition: 6	Leader: 174	Replicas: 174	Isr: 174
	Topic: cabinetBaseInfoQueue	Partition: 7	Leader: 175	Replicas: 175	Isr: 175

```

* topic: 主题
* partion 分区
* leader 负责读写的节点
* Replicas leader分区的副本
* Isr 同步分区
* broker.id  这里的值

---

> 2 查看消费者组列表及详情

```
kafka-consumer-groups --bootstrap-server 10.204.52.173:9092 --list
kafka-consumer-groups --bootstrap-server 10.204.52.173:9092 --describe --group resourcepool-cell-boxDetail-handle-group

```

查看指定group的详情如下

```
TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                                     HOST            CLIENT-ID
epspBizData     0          84              84              0               epspBizData_10.204.49.68_1-48913586-88cf-4696-a7d9-f8f3da8459bf /10.204.49.102  epspBizData_10.204.49.68_1
epspBizData     1          80              80              0               epspBizData_10.204.49.68_1-48913586-88cf-4696-a7d9-f8f3da8459bf /10.204.49.102  epspBizData_10.204.49.68_1
epspBizData     2          79              79              0               epspBizData_10.204.49.68_1-48913586-88cf-4696-a7d9-f8f3da8459bf /10.204.49.102  epspBizData_10.204.49.68_1
epspBizData     3          80              80              0               epspBizData_10.204.49.68_1-48913586-88cf-4696-a7d9-f8f3da8459bf /10.204.49.102  epspBizData_10.204.49.68_1
epspBizData     4          81              81              0               epspBizData_10.204.49.69_1-4dc4bcf1-9f2e-4254-9507-49dc89747a39 /10.204.49.103  epspBizData_10.204.49.69_1
epspBizData     5          79              79              0               epspBizData_10.204.49.69_1-4dc4bcf1-9f2e-4254-9507-49dc89747a39 /10.204.49.103  epspBizData_10.204.49.69_1
epspBizData     6          80              80              0               epspBizData_10.204.49.69_1-4dc4bcf1-9f2e-4254-9507-49dc89747a39 /10.204.49.103  epspBizData_10.204.49.69_1
epspBizData     7          86              86              0               epspBizData_10.204.49.69_1-4dc4bcf1-9f2e-4254-9507-49dc89747a39 /10.204.49.103  epspBizData_10.204.49.69_1

```

* CURRENT-OFFSET: group 消费者群组最近提交的偏移量
* LOG-END-OFFSET: broker 当前高水位偏移量，最近一个被读取消息的偏移量
* LAG： CURRENT-OFFSET 和 LOG-END-OFFSET 之间的差异

---

> 3 列出每个主题或broker的配置

```
kafka-configs --zookeeper 10.204.52.173:2181 --describe --entity-type topics --entity-name cabinetBaseInfoQueue
kafka-configs --zookeeper 10.204.52.173:2181 --describe --entity-type brokers

```

主题配置情况如下：

```
Configs for topic 'cabinetBaseInfoQueue' are 
Configs for topic 'test' are 
Configs for topic '__consumer_offsets' are segment.bytes=104857600,cleanup.policy=compact,compression.type=producers

```

* segment.bytes 单个片段的大小
* cleanup.policy 片段清理策略
* compression.type 压缩类型


> 4 生产者生产消息

```
#交互式
kafka-console-producer --broker-list 10.204.241.57:9092 --topic test

#从文件读取消息并发送
kafka-console-producer --broker-list 10.204.241.57:9092 --topic cabinetBaseInfoQueue < tmp

```

> 5 消费者消费消息

```
kafka-console-consumer --bootstrap-server 10.204.241.57:9092 --topic

```

> 6 创建主题

```
#指定分区数量和复制系数，复制系数决定分区副本的数量

kafka-topics --create --zookeeper 10.204.241.57:2181 --replication-factor 1 --partitions 1 --topic test

```
