## socket查询

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
