## 常用语法

### 1 点查询

look up on xx where xxx yield xxx as xxx

```
lookup on device where device.group == "default" yield device.productKey as productKey,device.type as type,device.deviceId as deviceId,device.group as group

```

### 2 统计子节点数量

GO xxx to xx steps from xxx over xxx yield xxx | group by $-.xxx, count(*)

```
GO 1 to 2 steps from xxx  over has_child  yield DISTINCT $$.device.deviceId as deviceId,has_child.createTime as createTime,$$.device.type as type,$$.device.productKey as productKey,$$.device.group as group | GROUP BY $-.productKey YIELD $-.productKey, count(*)

```
