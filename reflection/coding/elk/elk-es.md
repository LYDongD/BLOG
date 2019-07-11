## ELK-日志存储和索引

### 配置

/home/elk/config/es.yml

```
version: '2.0'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.5.1
    container_name: es
    environment:
      - cluster.name=es-cluster
      - node.name=es1
      - bootstrap.memory_lock=true
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g" 
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 4g
    ports:
      - 9200:9200
      - 9300:9300
    restart: always
    privileged: true

```

* es依赖jvm，需要分配特定的内存

**提高虚拟内存值**

max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]


* 进入sysctl.conf:    vim /etc/sysctl.conf
* 添加配置:    vm.max_map_count=655360
* 更新配置： sysctl -p 
* 重启ES


---

### 启动

使用docker-compose启动

```
docker-compose -f /home/elk/config/es.yml up -d

```

---

### 查看日志

```
tail -f `docker inspect --format='{{.LogPath}}' es`

```