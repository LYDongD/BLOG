## redis 配置

### 常见问题

> ERR This instance has cluster support disabled

redis server 需要开启集群模式, 找到redis.conf， 修改配置：

```
cluster-enabled yes

```

> ERR Client sent AUTH, but no password is set

redis server 需要设置密码，或客户端需要放弃提交密码

设置描眉，修改redis.conf:

```
requirepass xxx

```


