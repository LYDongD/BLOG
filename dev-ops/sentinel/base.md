## sentinel 接入文档

参考[sentinel官方文档](https://github.com/alibaba/Sentinel/wiki)

### 需求

为某个方法添加熔断机制，防止高峰时段该方法被频繁调用，占用过多资源，特别是数据库资源，导致其他核心业务不可用或性能受损。


### 熔断策略制定

**根据方法RT进行熔断**

通过分析该方法的执行时间，为方法设置一个RT阈值，超过阈值将熔断该方法，在有限时间窗口内让拒绝该方法调用。

#### 分析方法：

编写耗时分析脚本，对生产环境方法耗时日志进行统计分析。

#### 分析脚本

shell脚本

```
#!/bin/bash

LOG_FILE=/app/applogs/edmssvr/svrs.log
SYNC_FILE=/tmp/tmpSync.log

#fork sub shell to grep data
(grep "doBus:edcode" $LOG_FILE > $SYNC_FILE)

declare -A cost_dic
count=0
max=0
min=100000

#cal and save to dic
while read line
do
    line=${line}
    cost=$(echo ${line} | sed "s/.*cost=\([0-9]*\).*/\1/g" | bc)
    if test $[cost] -gt 2000; then
        let cost_dic[">2000ms"]+=1
    elif test $[cost] -gt 1000; then
        let cost_dic["1000-2000ms"]+=1
    elif test $[cost] -gt 500; then
        let cost_dic["500-1000ms"]+=1
    elif test $[cost] -gt 400; then
        let cost_dic["400-500ms"]+=1
    elif test $[cost] -gt 300; then
        let cost_dic["300-400ms"]+=1
    elif test $[cost] -gt 200; then
        let cost_dic["200-300ms"]+=1
    elif test $[cost] -gt 100; then
        let cost_dic["100-200ms"]+=1
    elif test $[cost] -gt 0; then
        let cost_dic["<100ms"]+=1
    fi

    if test $[cost] -gt $[max]; then
        let max=$cost
    elif test $[cost] -lt $[min]; then
        let min=$cost
    fi

    let ++count
done < $SYNC_FILE

#print dic
echo 总数：$count 最大耗时: $max ms 最小耗时: $min ms
echo 分布 数量 占比
for key in ${!cost_dic[*]}; do
    percent=$(echo "scale=2;${cost_dic[$key]}*100/$count" | bc)
    echo $key ${cost_dic[$key]} $percent%
done

#delete tmp file
rm $SYNC_FILE

```

#### 分析结论

对生产环境的日志文件进行分析，分析得出该方法调用耗时如下：

```
总数：39441 最大耗时: 1146 ms 最小耗时: 25 ms
分布 数量 占比
300-400ms 87 .22%
100-200ms 4285 10.86%
500-1000ms 29 .07%
<100ms 34527 87.54%
400-500ms 16 .04%
1000-2000ms 3 0%
200-300ms 494 1.25%

```

超过80%的执行时间集中在100ms以内，因此可以将100ms设置为RT的阈值，即1s内平均RT如果超过100ms，说明此时系统负载较高，资源不足，将进行熔断，并保持5s的熔断时间窗口

* RT = 100ms
* 时间窗户 = 5s
* 资源：box_used_rate_detail_syn


### 接入sentinel

#### 添加依赖

```
        <dependency>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-spring-context</artifactId>
            <version>0.3.1</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-annotation-aspectj</artifactId>
            <version>1.6.0</version>
        </dependency>
        <dependency>
            <groupId>com.fcbox</groupId>
            <artifactId>fcbox-sentinel-nacos-adapter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-dubbo-adapter</artifactId>
            <version>1.6.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-transport-simple-http</artifactId>
            <version>1.6.0</version>
            <exclusions>
                <exclusion>
                    <artifactId>fastjson</artifactId>
                    <groupId>com.alibaba</groupId>
                </exclusion>
            </exclusions>
        </dependency>

```

其中fcbox-sentinel-nacos-adapter传递依赖：

该项目依赖fastjson反序列拉取的动态规则配置，如果版本过低，会导致资源配置失效；需要升级为1.2.x

```
<dependencies>
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-cluster-client-default</artifactId>
            <version>${sentinel-version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-cluster-server-default</artifactId>
            <version>${sentinel-version}</version>
        </dependency>

        <!--传输数据给dashboard-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-transport-netty-http</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
            <version>${sentinel-version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-parameter-flow-control</artifactId>
            <version>${sentinel-version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-reactor-adapter</artifactId>
        </dependency>
    </dependencies>

```

### 切面配置

在配置文件app-core-context.xml中增加切面的spring bean配置

```
<bean id="sentinelResourceAspect" class="com.alibaba.csp.sentinel.annotation.aspectj.SentinelResourceAspect"/>

	<aop:config>
		<aop:pointcut expression="@annotation(com.alibaba.csp.sentinel.annotation.SentinelResource)" id="sentinelResourcePointcut"/>
        <aop:aspect ref="sentinelResourceAspect">
            <aop:around method="invokeResourceWithSentinel" pointcut-ref="sentinelResourcePointcut"/>
        </aop:aspect>
    </aop:config>

```

springboot配置切面的方式：

```
@Configuration
public class SentinelConfiguration {
    @Bean
    public SentinelResourceAspect sentinelResourceAspect() {
        return new SentinelResourceAspect();
    }
}

```


为格口明细同步方法增加sentinel注解，进行资源管控

```
 @SentinelResource(value = DegradeResource.BOX_USED_RATE_DETAIL_SYNC,
            blockHandler = "handleException",
            blockHandlerClass = {BoxUsedRateSyncDegradeHandler.class})
    public void doBus(final BoxUsedBean boxUsedBean) {
    
    }

```

当RT熔断规则触发时，该方法将被熔断，并回调blockHandlerClass的blockHandler方法

```
    public static void handleException(BoxUsedBean boxUsedBean, BlockException blockException){
        logger.error("格口明细同步方法被降级，拒绝处理， 参数:【{}】", JSON.toJSONString(boxUsedBean), blockException);
    }
```

#### 启动应用

启动应用时指定dashboard服务的tcp server socket唯一地址 **ip:port**, 并指定客户端端口和名称

```
-Dcsp.sentinel.dashboard.server=xxx:8080 
-Dproject.name=terminal-web 
-Dcsp.sentinel.api.port=8792

```

#### 控制台

启动应用后，可登陆dashboard控制台进行查看

* 必须触发资源，才能在dashboard查看相应监控点
* dashboard配置可接入持久化方案，例如nacos，可在nacos控制台查看规则
* sentinel通过引入依赖的方式快速支持dubbo，servlet等接口，如果对方法监控，需要通过api或注解切面实现

