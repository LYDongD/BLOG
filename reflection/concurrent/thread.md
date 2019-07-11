## 线程查看

> 常用命令

* pstree -p [pid] 进程和线程树
* top -Hp [pid] 线程资源消耗
* jstack [pid] 线程栈 -> thead dump

> 查看线程栈

```

//runnable
"i am thread 8" #18 prio=5 os_prio=31 tid=0x00007fd95288b800 nid=0x5e03 runnable [0x0000700004f07000]
   java.lang.Thread.State: RUNNABLE
	at com.liam.demo.thread.CheckThreadDemo$1.run(CheckThreadDemo.java:14)
	at java.lang.Thread.run(Thread.java:745)

//waiting
"i am thread 2" #12 prio=5 os_prio=31 tid=0x00007fcb19872000 nid=0x5303 in Object.wait() [0x0000700007d2e000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000079574f8b8> (a java.util.concurrent.locks.ReentrantLock)
	at java.lang.Object.wait(Object.java:502)
	at com.liam.demo.thread.CheckThreadDemo$1.run(CheckThreadDemo.java:23)
	- locked <0x000000079574f8b8> (a java.util.concurrent.locks.ReentrantLock)
	at java.lang.Thread.run(Thread.java:745)

//blocked
"i am thread 2" #12 prio=5 os_prio=31 tid=0x00007f82b5093800 nid=0x5303 waiting for monitor entry [0x00007000039ec000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.liam.demo.thread.CheckThreadDemo$1.run(CheckThreadDemo.java:22)
	- waiting to lock <0x000000079574f8b8> (a java.util.concurrent.locks.ReentrantLock)
	at java.lang.Thread.run(Thread.java:745)

```

**tip:**

创建线程时，设置可读性强的线程名称很有必要，利于排查问题

* waiting to lock xxx  等待锁，阻塞在某个锁
* locked xxx 获得锁xx
* waiting on xxx 等待在某个对象上， 通常是调用了Object.wait()


> 线程中断

* thread.interrupt(): 中断一个线程，设置线程的中断标志位为true
* thread.isInterrupted() 判断当前线程是否中断，注意，当线程中断异常InterruptedException抛出后会清空标志位，必要时需要重置标志位
* thread.interrupted() 清空当前线程中断标志位

使用范式:

1. 在执行核心逻辑前，先判断线程是否被中断，是则退出
2. 如果线程可等待，则捕获线程中断异常，必要时需要重置线程中断状态
3. 多数情况1和2配合使用

```
public static void main(String args[]) throws Exception{

        List<Thread> threadList = new LinkedList<>();
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(() -> {

                while (true) {
                    //if thread is interrupted, end the logic
                    if (Thread.currentThread().isInterrupted()) {
                        System.out.println("Interrupted!!");
                        System.out.println(Thread.currentThread().isInterrupted());
                        break;
                    }

                    //waiting
                    try {
                        Thread.sleep(3000);
                    }catch (InterruptedException ie) {
                        ie.printStackTrace();
                        //reset interrupt flag
                        Thread.currentThread().interrupt();
                    }
                    
                    //some logic
                    System.out.println("hello world");
                }
            });

            thread.start();
            threadList.add(thread);
        }

        for (Thread thread : threadList) {
            thread.interrupt();
        }
    }

```

** 总结 **

线程感知中断的两种方式

1. waiting的线程或阻塞在io的runnable线程：抛出InterruptedException异常
2. 正常runnable的线程： 通过主动调用thread.isInterrupted()探测

** 为什么不建议使用stop, suspend 和 resume() 等api **

不会释放锁，可能导致死锁或线程被永久冻结的情况
