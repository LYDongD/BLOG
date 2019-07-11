## 线程池

#### 线程池的功能

* 资源复用：避免重复创建消耗资源，减少性能开销
	* workers机制
	
* 资源限制：避免资源创建过度导致内存溢出
	* 队列，给定资源无法处理情况下使用队列缓冲
	* 饱和策略，队列饱和执行机制
	
#### 线程池源码分析

**1 使用独占锁ReentrantLock保证worker操作的原子性**

例如，添加worker

```
private boolean addWorker(Runnable firstTask, boolean core) {
	
	...
	
	    final ReentrantLock mainLock = this.mainLock;

            //创建worker，包括待执行任务和线程
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {
                //独占锁，同时只有一个线程执行
                mainLock.lock();
                try {
    
					...
                    //加入workers, 即添加任务
                    workers.add(w);

                    //更新当前线程池累计最大线程数
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;

                    //加入成功
                    workerAdded = true;
                  ...
                } finally {
                    //记得释放
                    mainLock.unlock();
                }
                
	...              

}


```

**2 优雅地关闭**

调用shutdown方法关闭线程池，清理线程，使进程能够正常退出:

* 拒绝加入新的任务
* 执行完旧的任务
* 中断所有线程

```

public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            //权限检查
            checkShutdownAccess();
            //设置当前线程池状态为SHUTDOWN，如果已经是SHUTDOWN则直接返回
            advanceRunState(SHUTDOWN);
            //中断所有线程：包括空闲的线程和执行任务的线程，如果线程未被终止，会导致进程(jvm)无法正常退出
            interruptIdleWorkers();
            //通知
            onShutdown(); // hook for ScheduledThreadPoolExecutor
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
    }


```

<font color=red> 如果要关闭进程，必须要调用线程池的shutdown清理池的所有线程，否则无法正常关闭 </font>

**最佳实践**

参考rocketmq, 用shutdownHook实现了所有资源的优雅关闭

* 为资源定义shutdown方法
* 在shutdownHook中优雅关闭资源

```
Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                try {
                    for (BrokerController brokerController : BROKER_CONTROLLERS) {
                        if (brokerController != null) {
                            brokerController.shutdown();
                        }
                    }

                    // should destroy message store, otherwise could not delete the temp files.
                    for (BrokerController brokerController : BROKER_CONTROLLERS) {
                        if (brokerController != null) {
                            brokerController.getMessageStore().destroy();
                        }
                    }

                    for (NamesrvController namesrvController : NAMESRV_CONTROLLERS) {
                        if (namesrvController != null) {
                            namesrvController.shutdown();
                        }
                    }
                    for (File file : TMPE_FILES) {
                        UtilAll.deleteFile(file);
                    }
                } catch (Exception e) {
                    logger.error("Shutdown error", e);
                }
            }
        });


```

**3 Worker继承AQS，实现了简单的不可重入的独占锁**

AQS的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态


```

 private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable

```

隐式的锁管理：通过status来表示节点同步状态：

* state > 0 获取了锁
* state = 0 释放了锁
* state = -1 禁止中断

worker做了简单的状态改变的封装：

```

public void lock()        { acquire(1); }
public boolean tryLock()  { return tryAcquire(1); }
public void unlock()      { release(1); } 
public boolean isLocked() { return isHeldExclusively(); }


```

执行worker的时候使用了隐式锁：

```
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
        	  //获取锁
            w.lock();

            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    //执行任务
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                
                //释放锁
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}

```

**4 用高低位来表示状态**

使用一个int变量表示线程池状态和线程数：

```

//使用一个int变量，高低位机制，存放2种信息：线程池状态(高3位)和线程个数(低29位)
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

    //线程数量掩码(去掉前3位表示状态，其余表示个数)
    private static final int COUNT_BITS = Integer.SIZE - 3;

    //线程最大个数
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;

    // Packing and unpacking ctl
    //获取高3位：运行状态
    private static int runStateOf(int c)     { return c & ~CAPACITY; }
    //获取低29位：线程数量
    private static int workerCountOf(int c)  { return c & CAPACITY; }


```	





	