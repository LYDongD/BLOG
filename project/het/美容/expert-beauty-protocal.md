### 洁面仪协议


#### 消息规范

消息规范指美容服务通过MQ发送的消息格式及包含的字段


```
{wrinkle=3, pouch=1, pore=2, acne=0, blackhead=1, dryOilType=1, userId=11230215}

userId 用户ID
wrinkle 皱纹 1无，2轻度，3重度，4重度
pouch  眼袋 1无眼袋，2轻微，3严重
acne 痘痘 0无痘痘，1有痘痘
pore 毛孔 1紧致，2轻度，3中度，4重度
blackhead 黑头 1无，2极少，3轻度，4中毒，5重度
dryOilType 水油肤质类型 1干性，2中性偏干，3中性，4混合性偏干，5混合性, 6混合型偏油, 7 油性皮肤

```

以上参数必传，如若没有，则值取-1

#### 请求方法

* POST
* contentType = application/x-www-form-urlencoded

#### 参数

* data 【String】 推荐配置数据，json字符串

```
https://[domain]?date=[json_str]
```

#### 输出字段

* userId【Long】用户ID
* productId 【Long】 产品型号ID 
* grade 【Long】 档位(1-4), 每个部位的档位相同
* workTime 【Long】 总时长(1-4min)
* timeProportion 【String】 时长比例, 例如2:1:1:3:3, 分别对应，额头：鼻子：下巴：左脸：右脸


#### 参数举例：

```
{
	'userId' : 1057,
	'productId' : 449,
	'grade' : 1,
	'workTime' : 2,
	'timeProportion' : '2:1:1:3:3'
}

```

#### 规则动作的URL配置(产品业务需要注意)

https://[domain]?productId=449&grade=1&workTime=2&timeProportion=2:1:1:3:3

--

### 彩光仪协议

#### 请求方法

* POST
* contentType = application/x-www-form-urlencoded

#### 参数

* data 【String】 推荐配置数据，json字符串

```
https://[domain]?date=[json_str]
```

#### 输出字段

* userId【Long】用户ID
* productId 【Long】 产品型号ID 
* workTime 【Long】 总时长(1-5min)
* timeProportion 【String】 时长比例, 例如2:1:1:3:3, 分别对应，额头：鼻子：下巴：左脸：右脸
* light 【Integer】 灯光，0-不开启灯光，1-蓝光， 2-红光，3-黄光
* leftFaceGrade 【Long】 左脸档位(1-4)
* rightFaceGrade 【Long】 右脸档位(1-4)
* foreheadGrade 【Long】 额头档位(1-4)
* noseGrade 【Long】 鼻子档位(1-4)
* chinGrade 【Long】 下巴档位(1-4)


#### 参数举例：

```
{
	'userId' : 1057,
	'productId' : 449,
	'workTime' : 5,
	'timeProportion' : '2:1:1:3:3',
	'light' : 2,
	'leftFaceGrade' : 2,
	'rightFaceGrade' : 3,
	'foreheadGrade' : 1,
	'noseGrade' : 4,
	'chinGrade' : 5
}

```

#### 规则动作的URL配置(产品业务需要注意)

```
https://www.baidu.com?productId=449&workTime=3
https://www.baidu.com?productId=449&light=2
https://www.google.com?productId=449&leftFaceGrade=3&rightFaceGrade=3
...
```
--


### API

#### 获取美容场景状态

```
/**
* 获取用户美容场景的状态
* @param userId 用户id
* @return 开关状态, -1 异常， 0 关 1 开 2 被强制关闭或从未开启
* @throws
*/
Integer getUserSceneStatusForBeauty(Integer userId);

```

#### 批量添加美容规则接口

* URL:
http://200.200.200.50/v1/web/expert/scene/addBeautyRules/v1.0

* 权限

```
调用前需要先登录专家系统获取权限，使用权限cookies进行调用

登录：http://200.200.200.50/manages

账号：hetceshi

密码：123456

```

* 参数


*url*： 规则动作请求的url,如http://200.200.200.50/v1/app/chairdressing/meimei/devicecofig/saveDeviceConfig，不包括参数

*productIdForCaiGuang*： 彩光仪的设备id, 填4859

*productIdForJieMian*： 洁面仪的设备id, 填4974

*timeProportion*： 面部时间比例，填2:1:1:3:3

*timestamp*: 当前时间戳，精确到ms

* 注意

1. 每次批量添加规则，都会删除以前的所有规则，并更新为新的规则
2. 每次批量添加规则，都会删除以前用户匹配的美容场景，需要重新开启场景





