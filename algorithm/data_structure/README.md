## 数据结构与实现

#### 从hash到B+对比总结

* hash  vs  tree
    * hash时间效率更高O（1）
    * tree适合有序数据处理，例如排序，max/min key, range select等
* BST vs RBT
    * RBT 避免tree 频繁 update 可能退化成链表
* AVL vs  RBT
    * RBT fix balance成本更低
    * RBT 不是严格平衡
* RBT vs skipList
    * skiplist 实现简单
    * skiplist 适合range select
    * RBT 占用内存少
* RBT vs BT
    * BT 多分叉，高度更小，大数据情况下查找更快
    * BT 适合磁盘数据，适用数据的局部性原理
* BT vs B+T
    * B+ 适合range select


#### 
