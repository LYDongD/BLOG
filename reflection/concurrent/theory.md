## 并发理论

> Amdahl比率：

并行化加速比公式

```
Tn = T1( F + (1 - F ) / n) => Sn = T1 / Tn =  1 / (F + 1/ (n * (1 - F)) )

```

* Sn: 加速比
* Tn： n个cpu执行的时间
* T1： 单个cpu执行的时间
* F： 串行比例
* 1 - F ： 并行比例

** 结论 **

1. 当n(cpu核心数) 趋于无穷时，加速比 = 1 / F
2. 加速比的极限瓶颈在于整个任务的串行执行比例：F
3. 尽量降低串行比例


> Gustafson 比率

并行化加速比公式

```
Sn =  (a + bn) / (a + b) =  F + n(1 - F) = n + F ( n - 1)

```

* 优化前执行时间 : a + bn
* 优化后: a + b
* 串行比例： F = a / (a + b)


** 结论 ** 

1. 当串行比例一定的情况下，加速比和cpu核心数成正比
2. 尽量增加cpu核心数

> JMM

JMM 描述了线程如何在内存和CPU之间进行交互，基于happens-before原则，提供了可见性和有序性的语义支持

* 例如volatile关键字，通过barrier指令，阻止重排序或刷新/冲刷cpu缓存等

** happens-before原则 ** 

* 单线程原则: 单线程，前面的操作happens before 后面的操作
* 锁原则: 锁释放前的操作 happens before 获取锁之后的操作
* volatile原则: volatile变量，写操作happens before 对此变量的读操作(任意操作)
* 传递性原则: A -> B, B -> C => A -> C
* 线程启动原则: A 调用 B.start(), A start之前的操作 happens before B 的任何操作
* 线程中断原则: A 调用 B.interrupt(), A interrupt之前的操作 happens before B 检测到中断后(主动/异常)的操作
* 线程等待原则: A 调用 B.join(), B 的操作happens before A join之后的任何操作
* 线程终止原则: 一个对象的初始化 happens before 该线程的finalize 调用
