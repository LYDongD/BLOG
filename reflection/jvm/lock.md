## 锁

### 查看偏向锁使用情况

* 使用synchronized 关键字时，处于性能考虑，不会立即使用互斥锁；而是采用锁动态升级机制：

```
偏向锁 -> 轻量级锁 -> 重量级锁(互斥锁)

```

1. 当同步块始终被一个线程执行时，持有偏向锁，不会执行lock/unlock等操作
2. 当同步块存在争用，可被多个线程执行，但是一般在不同时段执行的情况，升级为轻量级锁，采用CAS非阻塞式方式保证原子性
3. 当同步块存在争取，且争用激烈，超过某个阙值，升级为重量级锁，将阻塞获取锁失败的线程，涉及状态切换和状态转移


#### 查看偏向锁的使用情况

参数：

* -XX:+UnlockDiagnosticVMOptions
* -XX:+PrintBiasedLockingStatistics
* -XX:TieredStopAtLevel=1
* -XX:BiasedLockingStartupDelay=0

```
public class SynchronizeTest {

    static Lock lock = new Lock();
    static int count = 0;

    public static void foo() {
        synchronized(lock) {
            count++;
        }
    }

    //仅一个线程执行同步代码，查看偏向锁使用情况
    public static void main(String[] args) throws InterruptedException{
        //lock.hashCode();
        //System.identityHashCode(lock);
        for (int i = 0; i < 1000000; i++) {
            foo();
        }
    }

    static class Lock {
        //@Override
        //public int hashCode() {
        //    return 0;
        // }
    }

}

```

执行：

java -XX:+UnlockDiagnosticVMOptions -XX:+PrintBiasedLockingStatistics -XX:TieredStopAtLevel=1 -XX:BiasedLockingStartupDelay=0 com/liam/demo/jvm/SynchronizeTest

结果：

```
# total entries: 0
# biased lock entries: 1475406
# anonymously biased lock entries: 13056
# rebiased lock entries: 0
# revoked lock entries: 0
# fast path lock entries: 190720
# slow path lock entries: 1

```

总结：从biased lock entries 可以看出，对象采用了biased lock
