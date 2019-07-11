## 网络调试

* ifconfig 查看网络接口，ip/广播/mac地址，查看子网掩码
* host/nslookup 查看域名映射
    * cat /etc/resolv.conf 查看域名服务器
* traceroute 查看路由链路


```
##提取指定网络接口的IP地址
ifconfig eth0 | egrep -o "inet [^ ]*" | grep -o "[0-9.]*"

##查看数据包传输链路
traceroute www.baidu.com


```
