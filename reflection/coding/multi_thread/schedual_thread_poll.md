## 任务调度线程池

### 功能

* 单次延迟执行任务
* 周期性任务
	* 固定频率： fix-rate
	* 固定延迟时间: fix-delay

### 关键对象

* 延迟队列：DelayedWorkQueue
* 调度任务：ScheduledFutureTask extends FutureTask


### 源码分析

**1 区分周期性任务中的fix-rate和fix-delay**

* fix-rate: 执行方式为 initDelay + n * rate
	* 如果到达指定时间上一个任务未执行完，不会并发执行当前任务，而是延迟执行
* fix-delay: 执行方式为 initDelay + [task_end_time] + delay
	* 其中task_end_time为任务结束的时间

	
```

private void setNextRunTime() {
            long p = period;
            if (p > 0) //p>0, 属于fix-rate,固定频率，当前任务开始执行的时间基础上加上间隔时间即可
                time += p;
            else // p < 0， 属于 fix-delay, 固定延时，当前时间(任务结束的时间)加上延时时间即可
                time = triggerTime(-p);
        }
        
        
long triggerTime(long delay) {
        return now() +
            ((delay < (Long.MAX_VALUE >> 1)) ? delay : overflowFree(delay));
    }

```

**2 三种类型任务的执行方式**

根据属性period决定：

* period = 0：  一次性任务
* period > 0： fix-rate
* period < 0:  fix-delay


```
 public void run() {

            //是否仅执行一次
            boolean periodic = isPeriodic();
            
            //如果是runnin和shutdownOk，则不会取消该任务
            if (!canRunInCurrentRunState(periodic))
                cancel(false);
            else if (!periodic)
                //仅执行一次，调用父类执行
                ScheduledFutureTask.super.run();
            else if (ScheduledFutureTask.super.runAndReset()) { //定时执行
                setNextRunTime(); //设置下次执行的时间
                reExecutePeriodic(outerTask); //重新加入延时队列
            }
        }
        
```

period在初始化线程池时指定：

例如fix-rate

```

 public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit) {
        if (command == null || unit == null)
            throw new NullPointerException();
            
        if (delay <= 0)
            throw new IllegalArgumentException();

        //period = -delay，使 period < 0
        ScheduledFutureTask<Void> sft =
            new ScheduledFutureTask<Void>(command,
                                          null,
                                          triggerTime(initialDelay, unit),
                                          unit.toNanos(-delay));
        RunnableScheduledFuture<Void> t = decorateTask(command, sft);
        sft.outerTask = t;

        //添加任务到延时队列
        delayedExecute(t);
        return t;
    }



```

**3 使用CAS修改任务状态，确保线程安全**

```

 protected void set(V v) {

        //new -> completing -> normal 任务正常结束
        //同一个任务多次提交到线程池，任务共享state，因此修改时需要考虑线程安全
        if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
            outcome = v;
            
            //这里只有一个线程可以进入，因此不必使用CAS修改状态
            UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
            finishCompletion();
        }
    }


```


### 最佳实践

rocketmq 的 nameserver 服务（NamesrvController）：

* 10s一次(fix-rate)扫描失活的broker并清理
* 10min一次获取kv配置文件的内容并打印


```

//线程池仅使用一个线程执行
private final ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor(new ThreadFactoryImpl(
        "NSScheduledThread"));


```

```

//10s一次扫描并清理坏死的broker
        this.scheduledExecutorService.scheduleAtFixedRate(new Runnable() {

            @Override
            public void run() {
                NamesrvController.this.routeInfoManager.scanNotActiveBroker();
            }
        }, 5, 10, TimeUnit.SECONDS);


```




```

    //10 min 打印一次配置表
        this.scheduledExecutorService.scheduleAtFixedRate(new Runnable() {

            @Override
            public void run() {
                NamesrvController.this.kvConfigManager.printAllPeriodically();
            }
        }, 1, 10, TimeUnit.MINUTES);

```

**实现了线程池的安全关闭：**

```

 //安全关闭，清理通信服务，监控服务和线程池
    public void shutdown() {
    
        this.remotingServer.shutdown();
        this.remotingExecutor.shutdown();
        
       //安全关闭线程池
        this.scheduledExecutorService.shutdown();

        if (this.fileWatchService != null) {
            this.fileWatchService.shutdown();
        }
    }
```


