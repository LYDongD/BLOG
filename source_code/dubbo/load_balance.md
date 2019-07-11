## 负载均衡实现

### 四种策略

* Random LoadBalance 随机策略
* RoundRobin LoadBalance 轮询策略
* LeastActive LoadBalance 最少连接(调用)策略
* ConsistentHash LoadBalance 一致性hash策略

### 核心算法

####1 权重稀释算法

考虑到jvm启动的预热机制，对于刚启动的节点，希望适当稀释它的权重

* 实际权重 = (启动时间 / 预热时间) * 配置权重
* 启动时间越长，稀释比例越低，预热时间默认为10min


```
   //权重稀释算法：实际权重 = (启动时间 / 预热时间) * 配置权重
    static int calculateWarmupWeight(int uptime, int warmup, int weight) {
        int ww = (int) ((float) uptime / ((float) warmup / (float) weight));
        return ww < 1 ? 1 : (ww > weight ? weight : ww);
    }

```

####2 随机策略算法

随机算法和随机偏移量算法：

1 如果权重一致，索引为1-length直接的随机数(平均随机)

2 如果权重不一致，则计算总权重，不同索引的权重偏移量不同

3 基于总权重生成随机数，根据随机数在总权重的偏移量来计算索引

4 偏移量递减算法确定偏移所在索引(偏移<0时，落在该索引)

```

//随机偏移量算法选择节点处理加权随机策略
if (totalWeight > 0 && !sameWeight) {
    // If (not every invoker has the same weight & at least one invoker's weight>0), select randomly based on totalWeight.
    int offset = random.nextInt(totalWeight);
    // Return a invoker based on the random value.
    for (int i = 0; i < length; i++) {
        offset -= getWeight(invokers.get(i), invocation);
        if (offset < 0) {
            return invokers.get(i);
        }
    }
}

```

####3 轮询策略算法

* 平均轮询

通过自增序列和节点数量取模获取节点索引

* 加权轮询

1 依次遍历所有可用的节点，选择比权限基准线大的节点

2 每完成一轮迭代，在最大权重内不断提升权重基准线

3 随着迭代次数增加，权重基准线不断抬高，只有权重高的节点会被选择

4 到达最大权重后，重新执行上述过程


```

        //CAS自增序列号， 为每一个方法维护一个序列号，每次方法调用，方法的序列号就自增一次，轮询下一个节点
        AtomicPositiveInteger sequence = sequences.get(key);
        if (sequence == null) {
            sequences.putIfAbsent(key, new AtomicPositiveInteger());
            sequence = sequences.get(key);
        }

        //加权轮询
        if (maxWeight > 0 && minWeight < maxWeight) {

            AtomicPositiveInteger indexSeq = indexSeqs.get(key);
            if (indexSeq == null) {
                indexSeqs.putIfAbsent(key, new AtomicPositiveInteger(-1));
                indexSeq = indexSeqs.get(key);
            }
            length = nonZeroWeightedInvokers.size();

            //每迭代一轮更新一次当前权重(权重基准)，一轮内比当前权重大的节点可被选择
            //在一个maxWeight周期内，随着迭代轮增加，权重基准也抬高，只有高权重的节点会被选择
            while (true) {
                //迭代每个节点
                int index = indexSeq.incrementAndGet() % length;
                int currentWeight;
                if (index == 0) { //每迭代一轮更新一次当前权重，比当前权重大的节点可被选择
                    currentWeight = sequence.incrementAndGet() % maxWeight;
                } else {
                    currentWeight = sequence.get() % maxWeight;
                }


                if (getWeight(nonZeroWeightedInvokers.get(index), invocation) > currentWeight) {
                    return nonZeroWeightedInvokers.get(index);
                }
            }
        }

        // Round robin平均轮询
        return invokers.get(sequence.getAndIncrement() % length);

```


