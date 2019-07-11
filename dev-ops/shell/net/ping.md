## ping

* -c 指定分组包的数量
* 查看round-trip time:   min/average/max/bias

ping www.baidu.com -c 5

```
PING www.a.shifen.com (14.215.177.39): 56 data bytes
64 bytes from 14.215.177.39: icmp_seq=0 ttl=52 time=8.865 ms
64 bytes from 14.215.177.39: icmp_seq=1 ttl=52 time=8.454 ms
64 bytes from 14.215.177.39: icmp_seq=2 ttl=52 time=8.847 ms
64 bytes from 14.215.177.39: icmp_seq=3 ttl=52 time=20.845 ms
64 bytes from 14.215.177.39: icmp_seq=4 ttl=52 time=10.997 ms

--- www.a.shifen.com ping statistics ---
5 packets transmitted, 5 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 8.454/11.602/20.845/4.707 ms

```

#### 使用fping查看可联网主机

* -a 查看可用主机
* -u 查看不可用主机
* -g 按范围查看所有主机的连通性

```

#查看10.8.35.1 - 10.8.35.255范围内的可用主机
fping -a 10.8.35.1 10.8.35.255 -g

```
