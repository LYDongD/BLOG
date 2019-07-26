## ELK 日志搜索与分析

* 中心化管理
* 快速搜索
* 日志统计分析
* 业务监控


### 框架图

![ELK框架](https://github.com/LYDongD/graphic/blob/master/markdown/elk.png?raw=true)


### 组件功能说明

* filebeat, 日志收集组件，可定义多种输入源，例如日志文件
* redis，作为消息队列组件，缓存日志并进行流量削峰, 日志缓存在list(队列)
* logstash，日志集中处理组件，结合filter等插件对日志进行处理
* elasticsearch, 日志存储和索引组件，提供最基础的日志查找功能
* kibana, 用于展示日志和可视化的web服务

#### redis做消息队列的利与弊

* 具备通信(发布订阅)和队列能力
* 简单，可以处理百万级别的日活，性能上比不上kafka
* 不能保证消息可达，顺序等要求
* 不能完全保证消息持久化，具有消息丢失的风险
