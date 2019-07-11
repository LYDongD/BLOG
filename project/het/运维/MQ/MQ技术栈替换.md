#### MQ技术栈替换

#####需求

将项目中的消息中间件ActiveMQ替换成RocketMQ

#####测试点

**1 消息发送**

**1.1 消息发送到展厅**

```
topic: SMART_HOME_DATA

测试：动作执行后，需要将动作执行记录发送到展厅

期望：

1 在rocketmq后台可根据topic查看消息发送成功
2 后台可看到发送消息的日志：

send to Queue:clife.scene.issued.queue Data is :{"userId":11234774,"userSceneId":10668,"userSceneName":"睡眠场景","inputs":[{"iType":"USER_EVENT","conditionName":"睡眠状态","conditionValue":"是入睡","inputValue":"入睡"}],"outputs":[{"oType":"DEVICE_DATA","oo
utName":"明灯智能香薰机","outParams":[{"outParamName":"雾化","outParamValue":"关闭","configDataField":"mist","actionParamValue":"0"},{"outParamName":"亮度","outParaa
mValue":"0%","configDataField":"brightness","actionParamValue":"0"}]}]}


```

**1.2 消息发送到网关**

```
topic: PUSH_GATEWAY_TOPIC

测试: 场景发生变更后，需要将变更消息发送到网关

期望：在rocketmq后台可根据topic查看消息发送成功

注意：该功能由网关方面测试
```
---

**2 消息接收**

2.1 消费设备实时运行数据

```
topic: EXPERT_DEVICE_RUN_DATA_TOPIC

测试: 其他服务向网关发送设备类型数据，场景系统能够正常消费

开启场景 - 发送设备数据 - 专家系统接收数据并进行规则计算 - 计算成功则进行动作下发

期望：

1 后台打印日志：“收到消息xxxx”
2 规则计算，动作执行流程和以前一样保持正常

```
2.2 消费用户事件数据

```
topic: EXPERT_USER_EVENT_TOPIC

测试: 其他服务向网关发送用户事件数据，场景系统能够正常消费

例如美容规则：

开启场景 - 发送用户事件数据(肤质数据) - 专家系统接收数据并进行规则计算 - 计算成功则进行动作下发

期望：

1 后台打印日志：“收到消息xxxx”
2 规则计算，动作执行流程和以前一样保持正常

```

3 **兼容**

如果消息发送到ActiveMQ, 规则系统也能正常消费，即兼容两种消息模式

#### 测试工具

**1 调用接口发送mq消息**

**接口：**http://200.200.200.50/v1/web/expert/scene/sendMsg

请求头需要添加后台管理系统cookies

**参数：**

* destination【string】: 对应数据类型的topic
* data【string】： json数据
* type【number】: 1 发送到rocketmq， 2 发送到activemq
* timestamp【string】: 时间戳 

**data格式举例**：

```
//设备实数数据举例,必须提交deviceId或mac中的一个
{
    "data": "{\"sleepStatus\":1,\"breathRateException\":0}", 
    "mac": "405EE1900127", 
    "deviceId": 0
}
```
```
//用户事件举例
{
    "userId": 112, 
    "pore": 1
}
```

**2 后台日志**

路径：/www/logs/clife-business-scene-calculate-quartz

```
例如：

10:49:03.104 INFO  [ConsumeMessageThread_20] c.c.b.e.c.datasource.push.RealTimePush:92 - 收到消息：{"data":"{\"deviceId\":30001051,\"status\":nn nn
ull,\"newStatus\":null,\"source\":2,\"updateTime\":null,\"sleepStatus\":1,\"breathRateException\":0,\"heartRateException\":0,\"belong\":null}","mac":"405EE1900127","deviceId":0}
```

**3 rocketMQ 后台**

[rocketmq后台](http://10.8.9.22:12581/#/message)

