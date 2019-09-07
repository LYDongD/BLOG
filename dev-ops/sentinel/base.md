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

#### 配置nacos数据源

添加naco核心组件SentinelNacosAutoReader

该组件是对ConfigService的封装，负责与nacos建立连接并拉取或接收规则配置更新

* 初始化时主动pull一次nacos配置
* 启动监听器，当配置发生变化时，nacos主动推送到该服务

```
@Configuration
@PropertySources({@PropertySource(value="classpath:/config/nacos.properties")})
@EnableNacosConfig(globalProperties = @NacosProperties(serverAddr = "${nacos.config.server-addr}", namespace = "${nacos.config.namespace}"))
@Slf4j
public class NacoConfig {

    /**
     *  连接naco server 并监听配置更新
     */
    @Bean
    public SentinelNacosAutoReader sentinelNacosAutoReader() {
        return new SentinelNacosAutoReader();
    }
}

```

该组件从外部属性文件中加载属性，并通过@EnableNacosConfig设置到nacos的全局配置组件中


SentinelNacosAutoReader的具体实现：

```
public class SentinelNacosAutoReader {
    private static final Logger log = LoggerFactory.getLogger(SentinelNacosAutoReader.class);
    @NacosInjected
    private ConfigService configService;

    public SentinelNacosAutoReader() {
    }

    @PostConstruct
    private void post() {
        AbstractListener flowRuleListen = new AbstractListener() {
            public void receiveConfigInfo(String configInfo) {
                SentinelNacosAutoReader.this.loadFlowRule(configInfo);
            }
        };
        AbstractListener degradeRuleLisener = new AbstractListener() {
            public void receiveConfigInfo(String configInfo) {
                SentinelNacosAutoReader.this.loadDegradeRule(configInfo);
            }
        };
        AbstractListener systemRuleListener = new AbstractListener() {
            public void receiveConfigInfo(String configInfo) {
                SentinelNacosAutoReader.this.loadSystemRule(configInfo);
            }
        };
        AbstractListener paramFlowRuleListener = new AbstractListener() {
            public void receiveConfigInfo(String configInfo) {
                SentinelNacosAutoReader.this.loadParamFlowFule(configInfo);
            }
        };
        AbstractListener authorityRuleListener = new AbstractListener() {
            public void receiveConfigInfo(String configInfo) {
                SentinelNacosAutoReader.this.loadAuthorityRule(configInfo);
            }
        };

        try {
            SentinelNacosConsts consts = new SentinelNacosConsts();
            this.initLoadRules(consts);
            this.configService.addListener(consts.getFlowRuleFileName(), consts.getGroupName(), flowRuleListen);
            this.configService.addListener(consts.getDegradeRuleFileName(), consts.getGroupName(), degradeRuleLisener);
            this.configService.addListener(consts.getSystemRuleFileName(), consts.getGroupName(), systemRuleListener);
            this.configService.addListener(consts.getAutorityFileName(), consts.getGroupName(), authorityRuleListener);
            this.configService.addListener(consts.getParamFlowFileName(), consts.getGroupName(), paramFlowRuleListener);
            log.info("完成sentinel from nacos 配置的监听");
        } catch (NacosException var7) {
            log.error("sentinel配置监听失败", var7);
        }

    }

    private void loadFlowRule(String configInfo) {
        try {
            List<FlowRule> flowRuls = (List)JSON.parseObject(configInfo, new TypeReference<List<FlowRule>>() {
            }, new Feature[0]);
            FlowRuleManager.loadRules(flowRuls);
            log.info("flowRuleListen 加载规则：{}", configInfo);
        } catch (Exception var3) {
            log.error("cast configinfo error", var3);
        }

    }

    private void loadDegradeRule(String configInfo) {
        try {
            List<DegradeRule> Ruls = (List)JSON.parseObject(configInfo, new TypeReference<List<DegradeRule>>() {
            }, new Feature[0]);
            DegradeRuleManager.loadRules(Ruls);
            log.info("degradeRuleLisener 加载规则：{}", configInfo);
        } catch (Exception var3) {
            log.error("cast configinfo error", var3);
        }

    }

    private void loadSystemRule(String configInfo) {
        try {
            List<SystemRule> Ruls = (List)JSON.parseObject(configInfo, new TypeReference<List<SystemRule>>() {
            }, new Feature[0]);
            SystemRuleManager.loadRules(Ruls);
            log.info("systemRuleListener 加载规则：{}", configInfo);
        } catch (Exception var3) {
            log.error("cast configinfo error", var3);
        }

    }

    private void loadAuthorityRule(String configInfo) {
        try {
            List<AuthorityRule> Ruls = (List)JSON.parseObject(configInfo, new TypeReference<List<AuthorityRule>>() {
            }, new Feature[0]);
            AuthorityRuleManager.loadRules(Ruls);
            log.info("authorityRuleListener 加载规则：{}", configInfo);
        } catch (Exception var3) {
            log.error("cast configinfo error", var3);
        }

    }

    private void loadParamFlowFule(String configInfo) {
        try {
            List<ParamFlowRule> Ruls = (List)JSON.parseObject(configInfo, new TypeReference<List<ParamFlowRule>>() {
            }, new Feature[0]);
            ParamFlowRuleManager.loadRules(Ruls);
            log.info("paramFlowRuleListener 加载规则：{}", configInfo);
        } catch (Exception var3) {
            log.error("cast configinfo error", var3);
        }

    }

    private void initLoadRules(SentinelNacosConsts consts) {
        ExecutorService service = Executors.newSingleThreadExecutor(new ThreadFactory() {
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r);
                t.setName("init-sentinel-rule-thread");
                t.setDaemon(true);
                return t;
            }
        });
        service.submit(() -> {
            try {
                String flowRuleString = this.configService.getConfig(consts.getFlowRuleFileName(), consts.getGroupName(), 5000L);
                this.loadFlowRule(flowRuleString);
                String degradeRuleString = this.configService.getConfig(consts.getDegradeRuleFileName(), consts.getGroupName(), 5000L);
                this.loadDegradeRule(degradeRuleString);
                String systemRuleString = this.configService.getConfig(consts.getSystemRuleFileName(), consts.getGroupName(), 5000L);
                this.loadSystemRule(systemRuleString);
                String autorityRuleString = this.configService.getConfig(consts.getAutorityFileName(), consts.getGroupName(), 5000L);
                this.loadAuthorityRule(autorityRuleString);
                String paramFlowRuleString = this.configService.getConfig(consts.getParamFlowFileName(), consts.getGroupName(), 5000L);
                this.loadParamFlowFule(paramFlowRuleString);
            } catch (NacosException var7) {
                var7.printStackTrace();
            }

        });
    }

    public ConfigService getConfigService() {
        return this.configService;
    }
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

