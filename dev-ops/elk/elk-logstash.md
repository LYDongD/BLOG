## ELK - 日志处理和投递


### 配置

/home/elk/config/logstash/redis_logstash_json.conf

```

input {
    redis {
        host => ["127.0.0.1"]
        port => "6379"
        key => "filebeat"
        password => pss123er
        type => "clife"
        data_type => "list"
    }
}


filter {
  json {
    skip_on_invalid_json => true
    source => "message"
  }

}

output {
    stdout {}
    elasticsearch {
        hosts => ["localhost:9200"]
        index => "clife-%{+YYYY.MM.dd}"
    }
}

```

* 输入：redis
* 过滤：json插件
* 输出: elasticsearch

---

### 启动

```
docker run -d --net=host -v /etc/localtime:/etc/localtime \
    -v /home/elk/config/logstash/:/usr/share/logstash/pipeline/ \
    --name logstash docker.elastic.co/logstash/logstash:6.5.1

```

---

### 查看日志

```
tail -f `docker inspect --format='{{.LogPath}}' logstash `

```

**例如，因为field类型不匹配导致投递失败**

mapper_parsing_exception：failed to parse field [message] of type [text]

```
{
￼
    "log": "[2018-11-27T02:13:22,648][WARN ][logstash.outputs.elasticsearch] Could not index event to Elasticsearch. {:status=>400, :action=>[\"index\", {:_id=>nil, :_index=>\"clife-2018.11.27\", :_type=>\"doc\", :_routing=>nil}, #<LogStash::Event:0x76e3fe21>], :response=>{\"index\"=>{\"_index\"=>\"clife-2018.11.27\", \"_type\"=>\"doc\", \"_id\"=>\"mQnxUmcBj9-Q11-aRCNA\", \"status\"=>400, \"error\"=>{\"type\"=>\"mapper_parsing_exception\", \"reason\"=>\"failed to parse field [message] of type [text]\", \"caused_by\"=>{\"type\"=>\"illegal_state_exception\", \"reason\"=>\"Can't get text on a START_OBJECT at 1:342\"}}}}}
", 
    "stream": "stdout", 
    "time": "2018-11-27T02:13:22.649673897Z"
}

```

