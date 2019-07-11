## APIDoc 文档快速生成工具

#### APIDoc特性

* 支持接口分组，实现快速导航
* 支持接口多版本，可查看版本差异
* 支持文档客户端测试(待集成)

#### 环境准备

1 安装apidoc

```
npm install apidoc -g

```

#### 定义项目

在项目根目录下定义apidoc.json:

例如：{USER_HOME}/clife-bigdata/clife-bigdata-api/clife-bigdata-api-app/clife-bigdata-api-app-scene/trunk


```
{
  "name": "规则引擎APP服务",
  "version": "2.3.8",
  "description": "App接口文档",
  "title": "规则引擎",
  "url" : "https://dp.clife.net/v1/app/expert/"
}


```

### 编写注释

apidoc扫描指定注释，解析注解生成文档

注释用法参考：[apidoc](http://apidocjs.com/)

```
    /**
     * @apiDefine AuthParam
     * @apiParam {String} appId 应用ID
     * @apiParam {number} timestamp 时间戳
     * @apiParam {String} accessToken 访问凭证
     */

    /**
     * @api {get} userScene/sceneList/v1.1 获取场景模板列表
     * @apiName GetSceneList
     * @apiGroup SCENE
     * @apiVersion 1.1.0
     * @apiDescription 获取场景模板列表，不包含设备
     * @apiUse AuthParam
     * @apiParam {String} [experience] 是否体验场景（1-是；0-否）
     * @apiParam {String} paged 是否分页（true-分页；false-不分页）
     * @apiParam {number} [pageIndex] 当前第几页，分页时必填
     * @apiParam {number} [pageRows] 每页记录数，分页时必填
     *
     * @apiSuccessExample {json} Success-response:
     * HTTP/1.1 0 OK
     *
     * {
     *     "code": 0,
     *     "data": {
     *         "pager": {
     *             "totalRows": 34,
     *             "pageRows": 20,
     *             "pageIndex": 1,
     *             "paged": true,
     *             "defaultPageRows": 20,
     *             "totalPages": 2,
     *             "currPageRows": 20,
     *             "pageStartRow": 0,
     *             "pageEndRow": 19,
     *             "hasPrevPage": false,
     *             "hasNextPage": true
     *         },
     *         "list": [
     *             {
     *                 "sceneId": 169,
     *                 "sceneName": "起床-自然醒",
     *                 "pictureUrl": "http://200.200.200.58:8981/group2/M00/04/A9/yMjIOlexvx2AM6lMAAIntWFEf5s520.png",
     *                 "addedStatus": 0,
     *                 "summary": "当你睁开眼睛，阳光洒进卧室、轻柔的音乐、舒适的温度、可口的早餐都已经准备就绪。",
     *                 "animationUrl": null,
     *                 "heatRate": 0,
     *                 "vigorIndex": 0,
     *                 "updateTime": null,
     *                 "attributeTagList": [
     *                     {
     *                         "attributeTagId": 1,
     *                         "attributeTagName": "睡眠",
     *                         "attributeTagType": null,
     *                         "comments": null,
     *                         "creatorId": null,
     *                         "createTime": null,
     *                         "updateTime": null
     *                     }
     *                 ]
     *             }
     *         ]
     *     }
     * }
     *
     *
     * @apiErrorExample {json} Error-response:
     * HTTP/1.1  error
     * {
     * "code": 100010201,
     * "msg": "缺少参数",
     * "data": { }
     * }
     */


    /**
     * @api {get} userScene/sceneList/v1.2 获取场景模板列表
     * @apiName GetSceneList
     * @apiGroup SCENE
     * @apiVersion 1.2.0
     * @apiDescription 获取场景模板列表，包含设备
     * @apiUse AuthParam
     * @apiParam {String} [experience] 是否体验场景（1-是；0-否）
     * @apiParam {String} paged 是否分页（true-分页；false-不分页）
     * @apiParam {number} [pageIndex] 当前第几页，分页时必填
     * @apiParam {number} [pageRows] 每页记录数，分页时必填
     *
     * @apiSuccessExample {json} Success-response:
     * HTTP/1.1 0 OK
     *
     * {
     *     "code": 0,
     *     "data": {
     *         "pager": {
     *             "totalRows": 34,
     *             "pageRows": 20,
     *             "pageIndex": 1,
     *             "paged": true,
     *             "defaultPageRows": 20,
     *             "totalPages": 2,
     *             "currPageRows": 20,
     *             "pageStartRow": 0,
     *             "pageEndRow": 19,
     *             "hasPrevPage": false,
     *             "hasNextPage": true
     *         },
     *         "list": [
     *             {
     *                 "sceneId": 169,
     *                 "sceneName": "起床-自然醒",
     *                 "pictureUrl": "http://200.200.200.58:8981/group2/M00/04/A9/yMjIOlexvx2AM6lMAAIntWFEf5s520.png",
     *                 "addedStatus": 0,
     *                 "summary": "当你睁开眼睛，阳光洒进卧室、轻柔的音乐、舒适的温度、可口的早餐都已经准备就绪。",
     *                 "animationUrl": null,
     *                 "heatRate": 0,
     *                 "vigorIndex": 0,
     *                 "updateTime": null,
     *                 "attributeTagList": [
     *                     {
     *                         "attributeTagId": 1,
     *                         "attributeTagName": "睡眠",
     *                         "attributeTagType": null,
     *                         "comments": null,
     *                         "creatorId": null,
     *                         "createTime": null,
     *                         "updateTime": null
     *                     }
     *                 ],
     *                 "sceneDeviceVOList": [ //v1.2才返回设备列表
     *                     {
     *                         "deviceTypeId": 18,
     *                         "deviceSubTypeId": null,
     *                         "deviceTypeName": "床垫",
     *                         "pictureUrl": "http://200.200.200.58:8981/group2/M01/00/FE/yMjIOlW7SvCAAl80AAAFr3M0-r8048.png"
     *                     },
     *                     {
     *                         "deviceTypeId": 19,
     *                         "deviceSubTypeId": null,
     *                         "deviceTypeName": "LED灯",
     *                         "pictureUrl": "http://200.200.200.58:8981/group2/M01/00/FE/yMjIOlW7StGAJpFSAAAMJBI2Xl4178.png"
     *                     },
     *                     {
     *                         "deviceTypeId": 20,
     *                         "deviceSubTypeId": null,
     *                         "deviceTypeName": "加湿器",
     *                         "pictureUrl": "http://200.200.200.58:8981/group2/M00/00/FE/yMjIOlW7SwmAC0DhAAAJTnmEz7U005.png"
     *                     }
     *                 ]
     *             }
     *         ]
     *     }
     * }
     *
     *
     * @apiErrorExample {json} Error-response:
     * HTTP/1.1  error
     * {
     * "code": 100010201,
     * "msg": "缺少参数",
     * "data": { }
     * }
     */
    @CheckAuth(method = AuthMethod.GET)
    @ResponseBody
    @RequestMapping("/sceneList/{version:v\\d\\.\\d}")
    public Object sceneList(@PathVariable String version, Scene scene, Boolean paged, Pager pager, String accessToken, Integer appId) {

    }


    /**
     * @api {get} userScene/sceneDevices/v1.2 获取场景设备列表
     * @apiName GetSceneDeviceList
     * @apiGroup SCENE
     * @apiVersion 1.2.0
     * @apiDescription 获取场景模板关联的设备小类
     * @apiUse AuthParam
     * @apiParam {String} sceneId 场景ID
     *
     * @apiSuccessExample {json} Success-response:
     * HTTP/1.1 0 OK
     *
     * 	{
     * 	    "code": 0,
     * 	    "data": [
     * 	        {
     * 	            "deviceTypeId": null,
     * 	            "deviceSubTypeId": 14003,
     * 	            "deviceTypeName": "舒眠",
     * 	            "pictureUrl": "http://200.200.200.58:8981/group1/M00/0F/0A/yMjIOlo6EsuAE4aGAABJnPl5FZ0755.png"
     * 	        },
     * 	        {
     * 	            "deviceTypeId": null,
     * 	            "deviceSubTypeId": 5003,
     * 	            "deviceTypeName": "加湿器",
     * 	            "pictureUrl": "http://200.200.200.58:8981/group1/M00/0F/0A/yMjIOlo6EfOABMkMAABritkApDQ197.png"
     * 	        },
     * 	        {
     * 	            "deviceTypeId": null,
     * 	            "deviceSubTypeId": 11003,
     * 	            "deviceTypeName": "香薰机",
     * 	            "pictureUrl": "http://200.200.200.58:8981/group2/M01/0F/0A/yMjIOlo6HCCAQ7vBAABRvcvHtOQ853.png"
     * 	        },
     * 	        {
     * 	            "deviceTypeId": null,
     * 	            "deviceSubTypeId": 6001,
     * 	            "deviceTypeName": "睡眠监测器",
     * 	            "pictureUrl": "http://200.200.200.58:8981/group2/M00/0F/0A/yMjIOlo6FKqAa_UUAABH6sagOag676.png"
     * 	        }
     *     ]
     * 	}
     *
     *
     * @apiErrorExample {json} Error-response:
     * HTTP/1.1  error
     * {
     * "code": 100010201,
     * "msg": "缺少参数",
     * "data": { }
     * }
     */
    @CheckAuth(method = AuthMethod.GET)
    @ResponseBody
    @RequestMapping("/sceneDevices/{version:v\\d\\.\\d}")
    public Object sceneDevices(@PathVariable String version, Integer sceneId) {
        ...
        return JsonResult.success(voList);
    }


```

### 生成文档并发布到svn

vim clife-bigdata-api-app-scene/trunk/doc.sh


```
#!/bin/bash

#定义本地svn仓库地址
docSvnPath=/Users/lee/expert/
docPath=doc

#生成文档
apidoc -i ./ -o $docPath

#将文档提交到svn
cp -r doc $docSvnPath
cd $docSvnPath
svn add *
svn commit -m 'add new doc'

```

[文档地址](http://200.200.200.40/svn/repositories/server/wiki/clife/pages/expert/doc/index.html)
