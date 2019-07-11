## 逃逸分析优化

#### 原理

```

| 逃逸分析指的是对象的动态作用域分析
	| 全局逃逸
	| 参数级逃逸
	| 没有逃逸

| 逃逸分析优化(由JIT完成)
	| 如果一个对象没有发生逃逸（局部变量，且未发布出去）
		| 堆分配 -> 栈分配，不会竞争锁, 不需要GC，方法结束自动销毁
	| 如果一个对象只能被一个线程持有
		| 消除同步，锁消除
	| 如果一个对象的内存结构不是连续的
		| 部分保存在寄存器中，实现内存优化

```

#### 性能对比

执行以下程序，查看执行时间:

```
    public static void alloc() {
        byte[] bytes = new byte[2];
         bytes[0] = 1;
    }

    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 100000000; i++) {
            alloc();
        }

        long end = System.currentTimeMillis();
        System.out.println("耗时: " + String.valueOf(end - start) + "ms");
    }

```

耗时：8ms

##### 关闭逃逸分析后，输出如下：

-XX:-DoEscapeAnalysis -XX:+PrintGC

```
[GC (Allocation Failure)  16384K->536K(62976K), 0.0011630 secs]
[GC (Allocation Failure)  16920K->536K(62976K), 0.0009039 secs]
[GC (Allocation Failure)  16920K->504K(62976K), 0.0006959 secs]
[GC (Allocation Failure)  16888K->504K(79360K), 0.0007975 secs]
[GC (Allocation Failure)  33272K->488K(79360K), 0.0009008 secs]
[GC (Allocation Failure)  33256K->504K(110592K), 0.0009070 secs]
[GC (Allocation Failure)  66040K->448K(110592K), 0.0012194 secs]
[GC (Allocation Failure)  65984K->448K(176128K), 0.0003903 secs]
[GC (Allocation Failure)  131520K->448K(176128K), 0.0007156 secs]
[GC (Allocation Failure)  131520K->448K(254976K), 0.0015808 secs]
[GC (Allocation Failure)  210368K->448K(254976K), 0.0008193 secs]
[GC (Allocation Failure)  210368K->448K(380928K), 0.0007851 secs]
[GC (Allocation Failure)  336320K->448K(380928K), 0.0125635 secs]
[GC (Allocation Failure)  336320K->448K(392192K), 0.0004962 secs]
[GC (Allocation Failure)  347584K->448K(392192K), 0.0004795 secs]
[GC (Allocation Failure)  347584K->448K(392192K), 0.0004112 secs]
耗时: 1535ms


```

结论：

1 发生了GC, 说明对象在堆中分配
2 性能变差1000倍


##### 即使开启逃逸分析，如果关闭了标量替换，也无济于事

-XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-EliminateAllocations

```
[GC (Allocation Failure)  16384K->520K(62976K), 0.0026571 secs]
[GC (Allocation Failure)  16904K->536K(62976K), 0.0017814 secs]
[GC (Allocation Failure)  16920K->472K(62976K), 0.0012677 secs]
[GC (Allocation Failure)  16856K->504K(79360K), 0.0015036 secs]
[GC (Allocation Failure)  33272K->456K(79360K), 0.0028345 secs]
[GC (Allocation Failure)  33224K->504K(110592K), 0.0015363 secs]
[GC (Allocation Failure)  66040K->445K(110592K), 0.0027207 secs]
[GC (Allocation Failure)  65981K->445K(176128K), 0.0007060 secs]
[GC (Allocation Failure)  131517K->445K(176128K), 0.0026590 secs]
[GC (Allocation Failure)  131517K->445K(254976K), 0.0110181 secs]
[GC (Allocation Failure)  210365K->445K(254976K), 0.0209147 secs]
[GC (Allocation Failure)  210365K->445K(380928K), 0.0016749 secs]
[GC (Allocation Failure)  336317K->445K(380928K), 0.0016926 secs]
[GC (Allocation Failure)  336317K->445K(392192K), 0.0024312 secs]
[GC (Allocation Failure)  347581K->445K(392192K), 0.0020091 secs]
[GC (Allocation Failure)  347581K->445K(392192K), 0.0007193 secs]
耗时: 2477ms

```

结论:

1 逃逸分析优化建立在标量替换的基础上

#### 锁消除优化

-XX:+DoEscapeAnalysis -XX:+PrintGC 

```

/**
 *  逃逸分析优化demo
 */
public class EscapeDemo {

    //数组对象未逃逸, 如果开启逃逸分析，对象将在栈中分配，方法结束后被回收
    //-XX:-DoEscapeAnalysis关闭逃逸分析，默认开启
    //逃逸分析基于标量替换，
    public static void alloc() {
        byte[] bytes = new byte[2];
        synchronized (bytes) {
            bytes[0] = 1;
        }
    }

    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 100000000; i++) {
            alloc();
        }

        long end = System.currentTimeMillis();
        System.out.println("耗时: " + String.valueOf(end - start) + "ms");
    }
}

```

由于该锁对象每次都会被创建一个新的，相当于每个线程会持有一个新的锁对象，不会发生竞争，该锁会被优化消除

执行时间: 18ms

##### 关闭锁消除

-XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-EliminateLocks

```
[GC (Allocation Failure)  16384K->536K(62976K), 0.0034509 secs]
[GC (Allocation Failure)  16920K->536K(62976K), 0.0028587 secs]
[GC (Allocation Failure)  16920K->504K(62976K), 0.0052567 secs]
[GC (Allocation Failure)  16888K->488K(79360K), 0.0027440 secs]
[GC (Allocation Failure)  33256K->504K(79360K), 0.0021669 secs]
[GC (Allocation Failure)  33272K->504K(110592K), 0.0127228 secs]
[GC (Allocation Failure)  66040K->441K(110592K), 0.0027970 secs]
[GC (Allocation Failure)  65977K->441K(176128K), 0.0009319 secs]
[GC (Allocation Failure)  131513K->441K(176128K), 0.0015589 secs]
[GC (Allocation Failure)  131513K->441K(254976K), 0.0007677 secs]
[GC (Allocation Failure)  210361K->441K(254976K), 0.0012640 secs]
[GC (Allocation Failure)  210361K->441K(380928K), 0.0007695 secs]
[GC (Allocation Failure)  336313K->441K(364544K), 0.0008799 secs]
[GC (Allocation Failure)  320441K->441K(349696K), 0.0007965 secs]
[GC (Allocation Failure)  305593K->441K(335872K), 0.0007824 secs]
[GC (Allocation Failure)  291257K->441K(322048K), 0.0013744 secs]
耗时: 7695ms

```

结论:

1 关闭锁消除后性能下降接近1w倍
