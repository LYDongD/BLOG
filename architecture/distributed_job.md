## 分布式任务调度器

* 任务存储：redis
    * zset 
        * task_id -> trigger_time(score)
        * range: 0->now() get task list
    * hash 
        * task_id -> trigger_cron
        * get next trigger time by cront
* 创建任务
    * create two keys 
        * one for cron
        * one for time

* 检索任务并执行
    * init -> create/start thread -> poll tasks from redis -> check/reset time -> execute
    * can use SchedualThreadPoolExecutor on fixed rate 
    * reset/remove trigger time

* 任务治理
    * 任务未成功执行
        * 任务获取失败
        * 任务执行失败
            * 可以将未成功执行的任务写入到一个专门的key，例如retry_xxx
            * retry可自动或手动重试
* 并发安全
    * watch/unwatch -> lock key 
    * multi-exec -> transaction
        * if watch informed, transaction fail

* 库方案
    * 提供一个RedisTaskScheduler -> 实现分布式调度
        * 调度任务(创建任务)
        * 取消任务调度
        * 设置任务处理回调

    * 提供一个SchedulerFactory -> 创建并始化schedualer
    * 接入流程
        * 使用调度工厂创建调度器
        * 调度任务

* 性能
    * 1s 调度 10w+的 任务
