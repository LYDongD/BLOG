## 端口检测

#### lsof

list open file

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

```
