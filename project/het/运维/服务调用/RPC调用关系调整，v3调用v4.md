## RPC调用关系调整，v3调用v4

###规则引擎服务：

* A -> 规则配置开关系统
* B -> 规则执行调度系统

###需求背景

**当前v3规则引擎服务链路：**

```
上游服务(v3) -> A(v3) -> B(v3) -> 下游服务(v3)

A -> B 仅涉及一个接口：
UserSceneRunningService

```

由于v3下游服务停止, 但是上游服务还在使用，所有需要将链路调整为:

```
上游服务(v3) -> A(v3) -> B(v4) -> 下游服务(v4)

```
具体实施：

* 停止v3的B服务
* v3 A 调用 v4 B

### 实现方案

**1 多注册中心**

v3 A 注册zk的两个组，分别对应v3和v4不同环境下的两个组，接口UserSceneRunningService指向v4的组

```

<!--定义v4组-->
<dubbo:registry id="zookeeper-v4" protocol="zookeeper" address="${zookeeper.address}" group="${dubbo.group.v4}" />

<!--指向v4组-->
<dubbo:reference id="userSceneRunningService" interface="com.clife.bigdata.commons.scene.interfaces.UserSceneRunningService" check="false" retries="${dubbo.retry}" registry="zookeeper-v4" generic="true">
		<dubbo:method name="startUserScene" timeout="30000"/>
		<dubbo:method name="reStartUserScene" timeout="30000"/>
</dubbo:reference>

```

**2 泛化调用**

由于v3和v4中UserSceneRunningService的接口包名不一致，采用泛化调用的方式，无需在消费端引入接口

```
public class UserRunServiceTest{

GenericService genericService = (GenericService)context.getBean("userSceneRunningService");
boolean result = (boolean)genericService.$invoke("reStartUserScene", new String[] { "java.lang.Long" }, new Object[] { 116466L });
}

```

### 测试方案

通过**直连方式** RPC 调用v3的服务
(测试可通过python客户端进行调用）

* userSceneSwitchService

```
<dubbo:reference id="userSceneSwichTestService" interface="com.clife.commons.base.service.expert.userscene.UserSceneSwitchService"  check="false"  url="dubbo://200.200.200.77:20809"/>

```

**测试方法：**

* startUserScene 开启用户场景

```
  @Test
    public void callDierectV3CheckV4() {
        System.out.println("结果: " + userSceneSwichTestService.startUserScene(116466L));
    }
    
```

调用开启场景方法，去v4服务后台查看是否有开启场景的日志：

1. 登录v4服务器：200.200.200.47
2. 查看/www/logs/clife-business-sceneengine/normal_logs/normal.log
3. 根据用户场景id进行搜索, 能查到开启场景的日志












