## socket查询和参数改

一个连接即一个socket，由一个唯一的5元组唯一确定：

```
<sourceIp, sourcePort, tcp, destIp, destPort>

```


#### lsof

list open file， socket本质上是一个fd

```
lsof -i 列出所有端口，配合grep查看
lsof -i | grep ":[0-9]\+" -o | sort| uniq  提取端口号，排序并去重
lsof -i tcp 仅列出tcp端口
lsof -i:3306 查看特定端口的进程

```

#### netstat

```
#查看tcp连接及其端口
netstat -ntp

#查看进程占用的端口号
netstat -ltnp | grep 119809

—a/-t/-u/-x 分别对应所有/tcp/udp/unix连接类型

```

#### ss

```
#查看socket摘要
ss -s

##查看监听的tcp端口及其对应的进程
ss -tlp

-t/-u/-x 对应tcp/udp/unix 连接类型

```

---

### socket参数调整

> time_wait调优

time_wait是一个连接当中，主动发起关闭(调用close)的一方，在收到对方FIN包并回复ACK之后的状态，该状态将持续2MSL(1min)之久。

time_wait的作用是，避免新的连接复用或错误接收旧连接的数据，使得旧连接再2MSL时间内自然消亡。

[参考文章](https://zhuanlan.zhihu.com/p/40013724)

如何查看和设置socket的系统参数：

* sysctrl -a 查看所有socket参数
* sysctrl -w key=value 方式进行修改
* 直接修改/etc/sysctl.conf也可

关于time_wait的可调优参数：

```
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_timestamps = 1

```
