## ifconfig 配置网络连接

几个上网必须的关键参数：

* IFACE 网络接口
* IP ip地址
* MASK 子网掩码
* GW 网关
* HW 可选的mac地址

ifconfig可以通过root权限配置当前主机网络

```

#!/bin/bash
  
IFACE=en0
IP_ADDRESS=192.168.8.122
MASK=255.255.255.0
GW=192.168.8.1

#权限检查
if [ $UID -ne 0 ]; then
    echo "should run as root"
    exit 1
fi

#先关闭网络接口
/sbin/ifconfig $IFACE down

#配置网络接口
/sbin/ifconfig $IFACE $IP_ADDRESS netmask $MASK

#加入路由
route add default $GW

echo Successfully configured $IFACE

```
