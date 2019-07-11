## HTTP安全加密方案


#### 需求背景

现有的美容服务设备推荐是通过http请求发送的，该请求未进行任何安全防护，可能会导致以下问题：

* 信息泄露

抓包查看用户id,设备id，功能参数等敏感信息

* 请求伪造

伪造请求，故意触发设备

* 参数篡改

篡改设备功能参数，使设备触发异常

* 重放攻击

重复发送一个目的主机已接收过的包，使设备反复触发

为了保证设备推荐业务，包括未来通过http请求实现设备推荐的业务的安全性，需要在现有实现基础上增加安全防护

---

#### 需求

动作执行类型为2(即发送http请求)的动作，执行动作时，确保请求的安全性，即能够解决以下4个问题：

* 信息泄露
* 请求伪造
* 参数篡改
* 重放攻击


---

**方法：**

1. https加密报文
	
	* 更改请求协议为https

2. 使用数字签名防止请求伪造

	* 使用秘钥对参数进行md5加密，生成数字签名，签名不一致则认为是无效请求

3. 使用数字签名防止参数篡改

	* 使用秘钥对参数进行md5加密，生成数字签名，签名不一致则认为是无效请求

4. 使用时间戳确保请求仅一次有效，防止重放攻击

	* 请求时间不能超过30s，否则为无效请求


---

参考微信支付请求的安全机制

**实现方案：**

![http请求时序图](http://omdlmhd54.bkt.clouddn.com/http%E5%AE%89%E5%85%A8.png)


---

**步骤**


**1、签名算法**

签名生成的通用步骤如下：

第一步，设所有发送或者接收到的数据为集合M，将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串stringA。

特别注意以下重要规则：

◆ 参数名ASCII码从小到大排序（字典序）；

◆ 如果参数的值为空不参与签名；

◆ 参数名区分大小写；

◆ 传送的sign参数不参与签名，将生成的签名与该sign值作校验。

第二步，在stringA最后拼接上key得到stringSignTemp字符串，并对stringSignTemp进行MD5运算，再将得到的字符串所有字符转换为大写，得到sign值signValue。

◆ key设置路径：clife平台-->账户设置-->API安全-->密钥设置

举例：

假设传送的参数如下：

```
devicecId：	wxd930ea5d5a258f4f

userId：	10000100

switch：	1

color：	2

timeStamp: 1528083867111 //精确到ms

```

第一步：对参数按照key=value的格式，并按照参数名ASCII字典序排序如下：

stringA="color=2&devicecId= wxd930ea5d5a258f4f&switch=1&timeStamp=1528083867111&userId=10000100";

第二步：拼接API密钥：

```
//注：key为规则系统和美容服务共享的密钥key
stringSignTemp=stringA+"&key=RULEANDBEAUTY2018"

//注：MD5签名方式
sign=MD5(stringSignTemp).toUpperCase()="9A0A8659F005D6984697E2CA0A9CF3B7" 


```

最终得到最终发送的数据：

```
{
    "userId": "BeJson",
    "deviceId": "http://www.bejson.com",
    "switch": 1,
    "color": 2,
    "timeStamp": 1528083867111,
    "sign": "9A0A8659F005D6984697E2CA0A9CF3B7"
}
```

**2 秘钥key生成机制**

* 配置策略
 

```
1 开发者登录clife开放平台
2 登录秘钥生成模块生成或查看秘钥
3 开发者在代码中集成秘钥

如果开发者没有开发者账号，则默认开发者id为0

```

* 生成策略

32位字符，仅包含数字和大小写字符

```
 public static String getRandomString(int length) { //length表示生成字符串的长度
      String base = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz123456789";   //生成字符串从此序列中取
      Random random = new Random();
      StringBuffer sb = new StringBuffer();
      for (int i = 0; i < length; i++) {
          int number = random.nextInt(base.length());
          sb.append(base.charAt(number));
      }
      return sb.toString();
  }
```

* 存储策略

```
create table tb_api_key
(
	api_key_id bigint(20) NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT '主键',
	creator_id int(11) NOT NULL COMMENT '开发者ID',
	api_key varchar(32) NOT NULL comment 'api秘钥，32位字符，仅包含数字和大小写字母',
	create_time datetime NOT NULL DEFAULT CURRENT_TIMESTAMP comment '创建时间',
	update_time datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP comment '更新时间',
	status int(11) NOT NULL DEFAULT '1' comment '状态 -1 删除，0 异常 1 正常',
	key creator_id_idx(creator_id) comment '开发者ID索引',
    key api_key_idx(api_key) comment 'api_key索引'
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='api秘钥';

```

* api文档

[秘钥文档](http://200.200.200.40/svn/repositories/server/wiki/clife/index.html)

场景接口 - web接口 - 秘钥











