## 局部有序

#### 定义

局部有序： 多线程处理任务，并保证同一特定属性对应的任务串行

#### 举例

rocketmq的有序消息队列就支持局部有序性

#### 思路

* 用队列管理同一属性的所有任务，即属性任务
* 用线程池执行属性任务，线程池只会分配一个线程执行任务
* 属性任务循环消费队列任务，确保该属性下的所有任务串行消费

综上，保证只有一个线程处理一个特定属性的任务，不同属性的任务可以被多线程执行

#### 实现

**1 属性任务：用队列管理同一属性的任务**

```
/**
 *  在线程池中需要串行的属性任务
 *  不同线程执行不同的属性任务，使得同一属性的任务可以串行
 */
public class SerialThreadTask implements Runnable{

    //阻塞队列，用于维护该属性任务下的所有的任务,注意线程安全
    private BlockingQueue<Runnable> tasks = new LinkedBlockingQueue<>();

    //任务执行器
    private Executor executor;

    //是否该执行队列任务
    private AtomicBoolean shouldRun = new AtomicBoolean(false);


    public SerialThreadTask(Executor executor) {
        this.executor = executor;
    }

    //循环执行队列任务
    @Override
    public void run() {

        Runnable task = null;
        while (true){
            try {
                //取出任务执行
                task = tasks.poll(50, TimeUnit.MILLISECONDS);
                if (task != null){
                    task.run();
                }else {
                    //未取得任务，如果队列为空则停止执行
                    synchronized (this){
                        if (tasks.isEmpty() && shouldRun.compareAndSet(true, false)){
                            return;
                        }
                    }
                }
            }catch (InterruptedException ie){
                ie.printStackTrace();
            }

        }
    }


    /**
     * 添加任务并执行
     * @param task 任务
     */
    public void addTaskAndExecute(Runnable task){

        synchronized (this){
            tasks.add(task);

            //如果当前任务队列未执行，则执行任务
            if (shouldRun.compareAndSet(false, true)){
                executor.execute(this);
            }
        }
    }

```

**2 局部有序线程池：用线程池执行属性任务**

```
/**
 * 属性任务串行执行的线程池
 */
public class SerialThreadTaskExecutor {

    //执行属性任务队列的线程池
    private Executor executor;

    //维护所有属性任务
    private ConcurrentMap<Object, SerialThreadTask> serialThreadTaskQueueConcurrentMap = new ConcurrentHashMap<>();

    public SerialThreadTaskExecutor(Executor executor) {
        this.executor = executor;
    }

    /**
     * 串行执行该属性任务
     * 1 加入特定的属性任务队列
     * 2 让线程池执行该队列
     *
     * @param key  属性
     * @param task 该属性对应的任务
     */
    public void executeSerially(Object key, Runnable task) {

        //取出任务队列
        SerialThreadTask serialThreadTask = serialThreadTaskQueueConcurrentMap.get(key);

        //没有则新建一个并保存
        if (serialThreadTask == null) {
            serialThreadTask = new SerialThreadTask(executor);
            SerialThreadTask oldQueue = serialThreadTaskQueueConcurrentMap.putIfAbsent(key, serialThreadTask);
            //保存的时候如果已经被添加了，则使用已经有的
            if (oldQueue != null) {
                serialThreadTask = oldQueue;
            }
        }

        //添加任务并执行
        serialThreadTask.addTaskAndExecute(task);
    }


    public static void main(String args[]) {
        SerialThreadTaskExecutor serialThreadTaskExecutor = new SerialThreadTaskExecutor(Executors.newFixedThreadPool(4));
        for (int i = 0; i < 30; i++){

            final int order = i % 4;
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    System.out.println("当前任务序号: " + order + " 线程: " + Thread.currentThread());
                }
            };

            serialThreadTaskExecutor.executeSerially(order, runnable);
        }

    }

```

**3 测试**

```

//用4个线程的线程池执行属性任务
  public static void main(String args[]) {
        SerialThreadTaskExecutor serialThreadTaskExecutor = new SerialThreadTaskExecutor(Executors.newFixedThreadPool(4));
        for (int i = 0; i < 30; i++){

            final int order = i % 4;
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    System.out.println("当前任务序号: " + order + " 线程: " + Thread.currentThread());
                }
            };

            serialThreadTaskExecutor.executeSerially(order, runnable);
        }


// 任务序号即属性，统一属性的任务由同一个线程执行

当前任务序号: 1 线程: Thread[pool-1-thread-2,5,main]
当前任务序号: 0 线程: Thread[pool-1-thread-1,5,main]
当前任务序号: 2 线程: Thread[pool-1-thread-3,5,main]
当前任务序号: 0 线程: Thread[pool-1-thread-1,5,main]
当前任务序号: 3 线程: Thread[pool-1-thread-4,5,main]
当前任务序号: 1 线程: Thread[pool-1-thread-2,5,main]
当前任务序号: 2 线程: Thread[pool-1-thread-3,5,main]
当前任务序号: 3 线程: Thread[pool-1-thread-4,5,main]
当前任务序号: 1 线程: Thread[pool-1-thread-2,5,main]
当前任务序号: 2 线程: Thread[pool-1-thread-3,5,main]
当前任务序号: 3 线程: Thread[pool-1-thread-4,5,main]
当前任务序号: 1 线程: Thread[pool-1-thread-2,5,main]
当前任务序号: 2 线程: Thread[pool-1-thread-3,5,main]
当前任务序号: 3 线程: Thread[pool-1-thread-4,5,main]
当前任务序号: 1 线程: Thread[pool-1-thread-2,5,main]
当前任务序号: 3 线程: Thread[pool-1-thread-4,5,main]
当前任务序号: 2 线程: Thread[pool-1-thread-3,5,main]
当前任务序号: 3 线程: Thread[pool-1-thread-4,5,main]
当前任务序号: 1 线程: Thread[pool-1-thread-2,5,main]
当前任务序号: 3 线程: Thread[pool-1-thread-4,5,main]
当前任务序号: 2 线程: Thread[pool-1-thread-3,5,main]
当前任务序号: 1 线程: Thread[pool-1-thread-2,5,main]
当前任务序号: 2 线程: Thread[pool-1-thread-3,5,main]
当前任务序号: 1 线程: Thread[pool-1-thread-2,5,main]
当前任务序号: 0 线程: Thread[pool-1-thread-1,5,main]
当前任务序号: 0 线程: Thread[pool-1-thread-1,5,main]
当前任务序号: 0 线程: Thread[pool-1-thread-1,5,main]
当前任务序号: 0 线程: Thread[pool-1-thread-1,5,main]
当前任务序号: 0 线程: Thread[pool-1-thread-1,5,main]
当前任务序号: 0 线程: Thread[pool-1-thread-1,5,main]


```