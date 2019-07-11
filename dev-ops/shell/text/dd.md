## 测试文件的创建和磁盘io能力测试

### dd - 转换和拷贝文件

#### 用法

从输入读取指定大小的一个块，并拷贝到指定输出

* 用于备份数据，例如磁盘数据备份

```
//if 从file中读取输入而不是stdin（默认)
//of 输出到file而不是stdout(默认)
//bs 一次读/写的字节
//可以测试磁盘的读写速度
dd if=/dev/zero of=junk.data bs=1M count=1

//拷贝内存内容到硬盘，并指定块大小为1k
dd if=/dev/mem of=/root/mem.bin bs=1024

```
* 用于测试磁盘的读写能力

借助字符设备/dev/zero测试磁盘IO读写性能

```

1、测试磁盘写能力

# time dd if=/dev/zero of=/test.dbf bs=8k count=300000
因为/dev/zero是一个伪设备，它只产生空字符流，对它不会产生IO，所以，
IO都会集中在of文件中，of文件只用于写，所以这个命令相当于测试磁盘的写能力。

2、测试磁盘读能力

# time dd if=/dev/sdb1 of=/dev/null bs=8k
因为/dev/sdb1是一个物理分区，对它的读取会产生IO，/dev/null是伪设备，相当于黑洞，of到该设备不会产生IO，
所以，这个命令的IO只发生在/dev/sdb1上，也相当于测试磁盘的读能力。

3、测试同时读写能力

# time dd if=/dev/sdb1 of=/test1.dbf bs=8k
这个命令下，一个是物理分区，一个是实际的文件，对它们的读写都会产生IO（对/dev/sdb1是读，对/test1.dbf是写），
假设他们都在一个磁盘中，这个命令就相当于测试磁盘的同时读写能力

```

### 文件对比
