## 任务调度线程池

### 核心组件

* ScheduledFutureTask 定时/延时任务
* DelayedWorkQueue 管理延时任务的队列

### 核心功能

1 任务可比较大小

```
//compare task according to time and sequenceNumber 
public int compareTo(Delayed other) {
    if (other == this) // compare zero if same object
        return 0;
    if (other instanceof ScheduledFutureTask) {
        ScheduledFutureTask<?> x = (ScheduledFutureTask<?>)other;
        long diff = time - x.time;
        if (diff < 0)
            return -1;
        else if (diff > 0)
            return 1;
        else if (sequenceNumber < x.sequenceNumber)
            return -1;
        else
            return 1;
    }
    long diff = getDelay(NANOSECONDS) - other.getDelay(NANOSECONDS);
    return (diff < 0) ? -1 : (diff > 0) ? 1 : 0;
}


```

2 队列是优先级队列

* 优先级指的是最先执行的任务排在队首，反之在队尾
* 入队/出队都要进行排序
* 排序的依据是任务的比较器算法

```

/**
 * 入队
 * Sifts element added at bottom up to its heap-ordered spot.
 * Call only when holding lock.
 */
private void siftUp(int k, RunnableScheduledFuture<?> key) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        RunnableScheduledFuture<?> e = queue[parent];
        if (key.compareTo(e) >= 0)
            break;
        queue[k] = e;
        setIndex(e, k);
        k = parent;
    }
    queue[k] = key;
    setIndex(key, k);
}

```

3 优先级队列的数据结构是数组

* 数组实现堆
    * 完全二叉树
    * 父节点比子节点大
* 排序算法采用堆排序
    * 插入时，需要比较父子节点，找到插入位置
    * 删除时，需要比较父子节点，重新排序
    * 时间复杂度O（logn)







