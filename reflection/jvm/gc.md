## GC

### safe-point

> 什么是safe-point?

safe-point是java代码中，线程可能暂停的位置

处于safe-point的线程，不会破坏堆栈的状态，此时的线程状态可能是waiting/blocked/idel或正在执行安全的JNI等

> 为什么要引入safe-point机制？

某些操作需要避免其他线程破坏当前的堆栈状态，会发起stop-the-world请求，让其他线程进入sqfe-point

例如，GC，stack trace dump, 进入同步块

> 如何实现sqfe-point?

在适当的位置插入sqfe-point检查

例如：

* JIT编译执行的情况，在循环回边和调用返回的地方会插入sqfe-point检测
* 解释器执行的情况，可能每执行一条语句就检查一次

> safe-point对性能有何影响？

如果safe-point占用时间过长(通常是花了较长的时间才进入safe-point检查），会导致stop-the-world请求时间过长，即应用暂停时间过长

例如GC的pause time过长

> 如何查看safe-point的行为？

```
-XX:+PrintSafepointStatistics
–XX:PrintSafepointStatisticsCount=1
```

### GC回收算法

> 如何查看GC

```
-XX:+PrintGC 
-XX:+PrintGCDetails

[GC (Allocation Failure) [ParNew: 35072K->0K(39424K), 0.0004441 secs] 35698K->626K(126848K), 0.0004683 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

```





