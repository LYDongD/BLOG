## CPU资源消耗问题排查

### 如何排查CPU爆满问题

#### 1 排查cpu是否爆满以及爆满的进程

查看cpu的核心数

```
物理个数：cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
核心数：cat /proc/cpuinfo| grep "cpu cores"| uniq
逻辑cpu: cat /proc/cpuinfo| grep "processor"| wc -l
```

查看负载

```
//uptime / cpu cores 超过0.8应该引起重视
uptime

```

查看进程

top

#### 2 查看进程内线程cpu消耗情况

记录cpu消耗高的线程ID（10进制)

```
top -Hp $pid 

ps H -eo user,pid,tid,%cpu --sort=-%cpu | grep $pid | head

```

#### 3 堆栈转储

```
jstack -l 128547 > /tmp/jstack.dump

```

问题： 报错：well-known file is not secure，说明需要保证jtack执行用户和进程开启用户一致：

```
sudo jstack -l 128547 > /tmp/jstack.dump

```

问题： 找不到jtack命令，可通过

```
which java
ls -al xxx

根据软链找到java_home, 进一步找到jtack命令

```

### 4 堆栈分析

在堆栈文件内搜索指定的线程（线程号需要先转成16进制后进行搜索)

例如查找128559 -> 1f62f:

```
"Gang worker#0 (G1 Parallel Marking Threads)" os_prio=0 tid=0x00007f625c05f000 nid=0x1f62f runnable
```

这是一个gc标记线程

分析线程正在执行的方法，找到占用CPU的代码堆栈

