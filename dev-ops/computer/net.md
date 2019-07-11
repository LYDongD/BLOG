## 网络

### 如何查看机器是千兆还是万兆网卡？

> 查看网卡的速度

ifconfig 查看网络接口

ethtool etho 查看网卡接口的速度

```
Settings for eth0:
	Supported ports: [ TP ]
	Supported link modes:   1000baseT/Full 
	                        10000baseT/Full 
	Supported pause frame use: No
	Supports auto-negotiation: No
	Advertised link modes:  Not reported
	Advertised pause frame use: No
	Advertised auto-negotiation: No
	Speed: 10000Mb/s //万兆网络
	Duplex: Full
	Port: Twisted Pair
	PHYAD: 0
	Transceiver: internal
	Auto-negotiation: off
	MDI-X: Unknown

```


