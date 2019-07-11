## ELK-日志收集组件filebeat


### 配置

/home/elk/config/fielbeat.yml

```
filebeat.inputs:

- type: log

  paths:
    - /home/elk/logs/*.log


output.redis:
    hosts: ["127.0.0.1:6379"]
    password: "pss123er"
    key: "filebeat"
    db: 0

```

* 输入：log file
* 输出：redis

---

### 启动

```
docker run \
  --mount type=bind,source="/home/elk/config/filebeat.yml",target=/usr/share/filebeat/filebeat.yml \
  -v /etc/localtime:/etc/localtime \
  -v /home/elk/logs:/home/elk/logs/ \
  -v /home/elk/data/filebeat/logs:/usr/share/filebeat/logs\
  --net=host --name filebeat -d docker.elastic.co/beats/filebeat:6.5.1

```

**注意, 需要把host日志文件挂载到容器内，否则filebeat无法扫描到日志**

---

### 日志查看

查看docker输出日志

```
tail -f `docker inspect --format='{{.LogPath}}' filebeat`

```

**如何判断是否开始采集日志：**

查找harvester，如该日志，说明未扫描到日志文件

```
"filebeat\":{\"harvester\":{\"open_files\":0,\"running\":0}},\"libbeat\":{\"config\":{\"module\":{\"running\":0}},\"pipeline\":{\"clients\":1,\"events\":{\"active\":0}}}

```

如果开始扫描：

```
{
￼
    "log": "2018-11-27T09:38:41.382+0800	INFO	log/harvester.go:254	Harvester started for file: /home/elk/logs/test.log
", 
    "stream": "stderr", 
    "time": "2018-11-27T01:38:41.382330044Z"
}

```