## 索引

### 如何在创建索引时指定索引类型？

mysql支持两种索引：

* B+ tree
* hash

通过以下语句指定索引类型

```
create index dx_username using hash on tb_user(username); 

```

然而，innodb并不支持hash索引，如果坚持要指定hash类型，mysql会返回错误：

This storage engine does not support the HASH index algorithm, storage engine default was used instead.

### hash 索引的使用场景

hash索引用hash字典实现，读写速度都很快，但是无法实现范围查询，例如>, between等操作符相关的query

因此，hash索引适合key-value数据库，如果有人拿mysql做key-value存储的话

B+ tree 叶子节点是双向链表，利于范围查询和排序等


