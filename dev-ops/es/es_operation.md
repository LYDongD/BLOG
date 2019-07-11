## ES 相关api

[文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-query-lang.html)

### 集群健康:

curl -X GET "localhost:9200/_cat/health?v"

```
epoch      timestamp cluster      status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1556266388 16:13:08  routecluster yellow          2         2     36  18    0    0       25             0                  -                 59.0%

```

### 索引列表：

curl -X GET "10.204.49.59:9200/_cat/indices?v"

```
health status index       uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   cabinetbase 5eu7kRhCTfihAdzqPsifFQ   1   2      48199         1612      167mb         85.5mb
green  open   msg         weMNDkQbSsigZLHSjizStw   5   1          0            0      1.6kb           839b
yellow open   store_roo   7Cp-qEfTQEmZ4DOqs7roQA   9   3          0            0      2.8kb          1.4kb
yellow open   router      v3hvuEuNTJ-mHyqZKxza_g   3   3         96            1    866.4kb        428.3kb

```

### 节点列表：

curl -X GET "10.204.49.59:9200/_cat/nodes?v"

```
ip           heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
10.204.49.61           12          97   0    0.02    0.05     0.01 mdi       -      node-3
10.204.49.59            8          97  11    1.87    1.96     1.90 mdi       *      node-1

```

### 查询索引数据：

```

//默认返回前10条数据
curl -X GET "10.204.49.59:9200/cabinetbase/_search?pretty"

```
