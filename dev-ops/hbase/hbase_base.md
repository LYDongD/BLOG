## Hbase 基础总结

* NOSQL/列簇数据库
    * 不需要预定义schema（schema free)
    * 支持行级事务

* 业务场景
    * 积分流水，派寄件订单
    * 用户画像，信誉分
    * 广告 推送
    * 各维度格口统计，件量，营收统计
    * 柜机CPU，内存监控数据存储
    * OpenTSDB时序数据库存储

* 性能 (3个节点）
    * 读：6w/s 
    * 写： 4w/s

* 架构
    * client -> zk -> master
    * server -> DFS Client -> HDFS(DataNode)
    * HlReginServer
        * HLog (WAL机制，先写Log)
        * HRegion
            * Store
                * Mem Store (内存缓存，满后刷盘)
                * StoreFile
                    * DFS Client
                        * HDFS(DataNode)
* 集群
    * Hmaster
        * 连接Client, zk
        * 处理Client 请求指令（CRUD）
        * 管理多个RegionServer, 负载均衡等
    * 分区策略(hbase采用范围分区）
        * 范围分区(rowKey)
        * HASH分区
    * 强一致性（服务宕机策略）
        * 通过分布式log存储进行恢复
        * hmaster重新分配region

* Hbase读写流程
    * 连接zk，获取meta-region-server
    * 连接meta-region-server, 查询rowkey所在RS
    * 连接目标RS处理请求
    * 目标RS返回处理结果

一次读或写需要经历两次网络请求，如何保证性能？

	各个服务节点都进行本地缓存

hbase如何实现”select *from user where name like ‘%xx%’ ” ？

	使用Filter过滤器，不过性能比较差，hbase不适合过滤查询，适合主键或范围查询
	
* 表设计
    * 定制属性
        * SPLITS 分区，例如用户表根据手机号尾号进行分区
        * COMPRESSION，支持算法
    * rowkey(内部索引)
        * 设计原则
            * 足够散列
            * 方便查询
            * 尽可能端
    * 积分流水的Rowkey设计
        * user_type + user_id + op_time + appid + 3位随机数
* 二级索引
    * 使用外部存储实现二级索引 -> 一级索引rowkey
        * phoneix/es等
    * 使用内存hash保存二级索引映射
* sequenceId（region级别自增)
    * HLog通过sequenceId和原子写，实现行级事务实现
        * ACID(写提交级别的事务实现）
            * 通过队列存储版本，readpoint指向事务提交的版本
            * 读数据只能读取readPoint之前的版本
    * 恢复region时通过sequenceId实现
* HFile
    * rowkey 范围定位region
    * block 树查找
    * bloom filter 忽略不存在的key
    * 在找到的block进行顺序扫描
* 文件合并
    * 小文件  -> 合并大文件 -> 排序 -> 减少扫描的次数
* Copprocessor
    * Observer
    * Endpoint

