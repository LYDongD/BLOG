## 场景-开放平台MQ通信协议

### 队列名

* clife.open.push.gateway.queue


### 通信协议

* 数据格式： TextMessage
* 说明：发送的数据为json字符串，结构如下
* 举例：


```
{ 
	 "userId": 100, 
	 "updateTime": 1508829686000,    
	 "actionType": 3,  // 动作类型，3 增加或更新 4 删除
     "data": {
    	 "userSceneId": [7666], 
    	 "sceneId": 232, 
   	 	 "userId": 100, 
   		 "createTime": 1504943061000, 
   		 "updateTime": 1508829686000, 
   		 "enableTime": 1509526926000, 
   		 "runStatus": 1,  //运行状态， 0 未启动 1 已开启
    	 "validity": 1,  //有效性, 0 无效 1 有效
        "rules": [
            {
                "index": 0, 
                "conditions": [
                    {
                        "conditionTypeKey": null, 
                        "prefix": "", 
                        "expression": "sleepStatus==1", 
                        "suffix": "&&", 
                        "macAddress": "A20AFAB839CDFD5CC51B377CDDA357FC", 
                        "runDataField": "sleepStatus"
                    }
                ], 
                "actions": [
                    {
                        "macAddress": "CE539440BE2DD1C295F47AD2EB6879B9", 
                        "productId": 1410, 
                        "delayTime": null, 
                        "actionItems": [
                            {
                                "configDataField": "lightness", 
                                "actionParamValue": "20"
                            }, 
                            {
                                "configDataField": "controlNumber", 
                                "actionParamValue": "1"
                            }, 
                            {
                                "configDataField": "lightColorR", 
                                "actionParamValue": "255"
                            }, 
                            {
                                "configDataField": "lightColorB", 
                                "actionParamValue": "0"
                            }, 
                            {
                                "configDataField": "lightingPatternNumber", 
                                "actionParamValue": "1"
                            }, 
                            {
                                "configDataField": "lightSwitch", 
                                "actionParamValue": "1"
                            }, 
                            {
                                "configDataField": "lightColorG", 
                                "actionParamValue": "0"
                            }
                        ]
                    }, 
                    {
                        "macAddress": "CE539440BE2DD1C295F47AD2EB6879B9", 
                        "productId": 1410, 
                        "delayTime": 10, 
                        "actionItems": [
                            {
                                "configDataField": "speakerSoundNumber", 
                                "actionParamValue": "67"
                            }, 
                            {
                                "configDataField": "speakerPlayStatus", 
                                "actionParamValue": "1"
                            }, 
                            {
                                "configDataField": "speakerOperate", 
                                "actionParamValue": "0"
                            }, 
                            {
                                "configDataField": "speakerSoundSource", 
                                "actionParamValue": "1"
                            }, 
                            {
                                "configDataField": "speakerVolume", 
                                "actionParamValue": "40"
                            }, 
                            {
                                "configDataField": "speakerMode", 
                                "actionParamValue": "0"
                            }, 
                            {
                                "configDataField": "controlNumber", 
                                "actionParamValue": "2"
                            }, 
                            {
                                "configDataField": "lightSwitch", 
                                "actionParamValue": "1"
                            }
                        ]
                    }
                ]
            }, 
            {
                "index": 24, 
                "conditions": [
                    {
                        "conditionTypeKey": null, 
                        "prefix": "", 
                        "expression": "hourOfDay>18", 
                        "suffix": "||", 
                        "macAddress": null, 
                        "runDataField": "hourOfDay"
                    }, 
                    {
                        "conditionTypeKey": null, 
                        "prefix": "", 
                        "expression": "hourOfDay<7", 
                        "suffix": "||", 
                        "macAddress": null, 
                        "runDataField": "hourOfDay"
                    }
                ], 
                "actions": [
                    {
                        "macAddress": "994CA7ED2A572F1BF5873CEF272DAC21", 
                        "productId": 1715, 
                        "delayTime": null, 
                        "actionItems": [
                            {
                                "configDataField": "ctram", 
                                "actionParamValue": "0"
                            }
                        ]
                    }
                ]
            }
        ]
    }
}

```

