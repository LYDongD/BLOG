##ELK-日志搜索和监控

### 配置

config/kibana.yml

```
elasticsearch.url: "http://120.78.142.82:9200"

```

### 启动

```
cd bin/
./kibana
```

### 应用

#### 监控

添加sentinl插件，参考(https://sentinl.readthedocs.io/en/latest/#watcher-life-cycle)

##### input

es检索指定条件的数据, 可在“开放工具”通过search api进行调试

```

{
  "search": {
    "request": {
      "index": [
        "clife-*"
      ],
      "body": {
        "size": 1,
        "query": {
          "bool": {
            "must": [
              {
                "range": {
                  "@timestamp": {
                    "gte": "now-5m",
                    "lte": "now"
                  }
                }
              },
              {
                "term": {
                  "jsonMsg.comment.keyword": {
                    "value": "设备指令下发失败"
                  }
                }
              }
            ]
          }
        }
      }
    }
  }
}

```

**查询结果提取语法, 可在transforms, actions环节使用**


* To load all of the search hits into an email body, use **payload.hits.**
* To reference the total number of hits, use **payload.hits.total.**
* To access a particular hit, use its zero-based array index. For example, to get the third hit, use **payload.hits.hits.2.**
* To get a field value from a particular hit, use **payload.hits.hits.<index>.fields.<fieldname>.** For example, to get the message field from the first hit, use **payload.hits.hits.0.fields.message.**


#### Condition

```
{
  "script": {
    "script": "payload.hits.total > 0"
  }
}

```

#### action

选择webHook，发送报警信息到钉钉, 消息格式为markdown

[钉钉webhook](https://open-doc.dingtalk.com/docs/doc.htm?treeId=257&articleId=105735&docType=1)

```
HEADERS:

{
  "Content-Type": "application/json"
}

Body:

{
     "msgtype": "markdown",
     "markdown": {
     "title" : "设备指令下发失败", "text" : "## 指令下发失败 \n * 用户ID：{{payload.hits.hits.0._source.jsonMsg.info.userId}}\n * 用户场景ID：{{payload.hits.hits.0._source.jsonMsg.info.userSceneId}}\n * mac地址: {{payload.hits.hits.0._source.jsonMsg.info.mac}}\n * 指令: {{payload.hits.hits.0._source.jsonMsg.info.updateFlag}}\n * 错误信息: {{payload.hits.hits.0._source.stackTrace}}\n[点击搜索信息详情](http://10.12.9.211:8880/app/kibana)"
     },
     "at": {
         "isAtAll":true 
     }
 }

```

#### 效果

钉钉收到消息

![钉钉](https://github.com/LYDongD/graphic/blob/master/markdown/monitor_ding.png?raw=true)


#### 完整监控

```
{
  "actions": {
    "钉钉通知": {
      "throttle_period": "0h0m0s",
      "webhook": {
        "method": "POST",
        "host": "oapi.dingtalk.com",
        "port": 443,
        "proxy": false,
        "path": "/robot/send?access_token=e012361aded036f69fbaa4a1bbb9b7cff9b25978e8a96672dcb67891c6b5057b",
        "body": "{\n     \"msgtype\": \"markdown\",\n     \"markdown\": {\n     \"title\" : \"设备指令下发失败\", \"text\" : \"## 指令下发失败 \\n * 环境：{{payload.hits.hits.0._source.tags.0}}\\n * 用户ID：{{payload.hits.hits.0._source.jsonMsg.info.userId}}\\n * 用户场景ID：{{payload.hits.hits.0._source.jsonMsg.info.userSceneId}}\\n * mac地址: {{payload.hits.hits.0._source.jsonMsg.info.mac}}\\n * 指令: {{payload.hits.hits.0._source.jsonMsg.info.updateFlag}}\\n * 错误信息: {{payload.hits.hits.0._source.stackTrace}}\\n[点击搜索信息详情](http://10.12.9.211:8880/app/kibana)\"\n     },\n     \"at\": {\n         \"isAtAll\":true \n     }\n }",
        "priority": "high",
        "use_https": true,
        "headers": {
          "Content-Type": "application/json"
        }
      }
    }
  },
  "input": {
    "search": {
      "request": {
        "index": [
          "clife_bigdata-*"
        ],
        "body": {
          "size": 1,
          "query": {
            "bool": {
              "must": [
                {
                  "range": {
                    "@timestamp": {
                      "gte": "now-1m",
                      "lte": "now"
                    }
                  }
                },
                {
                  "term": {
                    "jsonMsg.traceTopic.keyword": "规则监控-指令下发"
                  }
                },
                {
                  "term": {
                    "log_level.keyword": "ERROR"
                  }
                }
              ]
            }
          }
        }
      }
    }
  },
  "condition": {
    "script": {
      "script": "payload.hits.total > 0"
    }
  },
  "transform": {},
  "trigger": {
    "schedule": {
      "later": "every 1 minutes"
    }
  },
  "disable": false,
  "report": false,
  "title": "设备指令发送失败告警",
  "save_payload": false,
  "spy": false,
  "impersonate": false
}

```
