## MQTT topic

### topic的匹配模式有哪些？

* 用"*"实现多层匹配
* 用+实现单层匹配

参考[topic的匹配模式](https://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices/)

### 如何保证topic变更及时更新

关闭连接前，使用unsbscribe/disconnect从broker移除clientId和topic之间的映射
