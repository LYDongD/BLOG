## 原子变量和互斥锁实现共享变量性能对比

### 实现机制

count++ 属于read-then-write的竞态模式，会产生线程安全问题

* atomic原子变量通过cas和volatile机制保证共享变量自增的数据一致性
* synchronize通过互斥锁确保线程安全

### 性能对比测试

调度10个线程，每个线程执行1w次count的自增操作，等待所有线程执行完毕后，查看共享变量的值

> 未使用atomic或锁的情况下，最后读取的值<100000, 数据不一致

> 采用synchronize

```
package com.liam.demo.thread;

import java.util.concurrent.CountDownLatch;

public class SynchronizeDemo {

    private int count = 0;

    private CountDownLatch threadCount;

    private final Object lock = new Object();

    private static int totalElapsedTime;

    public void testSynCountIncrement() throws Exception{
        long start = System.currentTimeMillis();

        //模拟10个线程，每个线程执行10000次，对共享变量count自增，期望最后count的值为100000
        threadCount = new CountDownLatch(10);
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {

                    for (int i = 0; i < 10000; i++) {
                        synchronized (lock) { //互斥锁，保证线程安全
                            count++;
                        }
                    }
                    threadCount.countDown();
                }
            });

            thread.start();
        }

        //等待所有线程执行完毕
        threadCount.await();

        //读取共享变量的值
        System.out.println("共享变量count: " + count);

        long end = System.currentTimeMillis();
        System.out.println("单次耗时: " + String.valueOf((end - start)) + "ms");

        totalElapsedTime += (end - start);
    }

    public static void main(String[] args) throws Exception{

        int times = 10;

        for (int i = 0; i < times; i++ ){
            SynchronizeDemo synchronizeDemo = new SynchronizeDemo();
            synchronizeDemo.testSynCountIncrement();
        }

        System.out.println("平均耗时: " + String.valueOf((totalElapsedTime / times)) + "ms");
    }

}


```

结果如下:

```
共享变量count: 100000
单次耗时: 36ms
共享变量count: 100000
单次耗时: 20ms
共享变量count: 100000
单次耗时: 10ms
共享变量count: 100000
单次耗时: 7ms
共享变量count: 100000
单次耗时: 6ms
共享变量count: 100000
单次耗时: 6ms
共享变量count: 100000
单次耗时: 5ms
共享变量count: 100000
单次耗时: 7ms
共享变量count: 100000
单次耗时: 6ms
共享变量count: 100000
单次耗时: 7ms
平均耗时: 11ms

```

> 使用atomic机制

```
package com.liam.demo.thread;

import java.sql.Timestamp;
import java.util.Date;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicDemo {

    private AtomicInteger count = new AtomicInteger(0);

    private CountDownLatch threadCount;

    private static int totalElapsedTime;

    private void testAtomicCountIncrement() throws Exception{

        long start = System.currentTimeMillis();

        //模拟10个线程，每个线程执行10000次，对共享变量count自增，期望最后count的值为100000
        threadCount = new CountDownLatch(10);
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {

                    for (int i = 0; i < 10000; i++) {
                        count.incrementAndGet();
                    }
                    threadCount.countDown();
                }
            });

            thread.start();
        }

        //等待所有线程执行完毕
        threadCount.await();

        //读取共享变量的值
        System.out.println("共享变量count: " + count.get());

        long end = System.currentTimeMillis();
        totalElapsedTime += (end - start);
        System.out.println("单次耗时: " + String.valueOf((end - start)) + "ms");
    }

    public static void main(String[] args) throws Exception{

        int times = 10;

        for (int i = 0; i < times; i++ ){
            AtomicDemo atomicDemo = new AtomicDemo();
            atomicDemo.testAtomicCountIncrement();
        }

        System.out.println("平均耗时: " + String.valueOf((totalElapsedTime / times)) + "ms");
    }

}

```

结果如下:

```
共享变量count: 100000
单次耗时: 14ms
共享变量count: 100000
单次耗时: 7ms
共享变量count: 100000
单次耗时: 5ms
共享变量count: 100000
单次耗时: 2ms
共享变量count: 100000
单次耗时: 3ms
共享变量count: 100000
单次耗时: 1ms
共享变量count: 100000
单次耗时: 4ms
共享变量count: 100000
单次耗时: 2ms
共享变量count: 100000
单次耗时: 2ms
共享变量count: 100000
单次耗时: 3ms
平均耗时: 4ms

```

#### 结论

atomic机制比synchronize互斥锁的方式要快一倍，总的来说，互斥锁的性能是比较差的
