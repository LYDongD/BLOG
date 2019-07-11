## mvn多环境构建

**1 用目录区分环境**

```

| src/main/resources
	|local
		| config.properties
	|dev
		| config.properties
	|product
		| config.properties
		
```

**2 在项目pom.xml中配置profile环境**

```

<profiles>
    <!--本地环境-->
    <profile>
        <id>local</id>
        <properties>
            <profiles.active>local</profiles.active>
        </properties>
        
        <!-- 设置默认激活这个配置 -->
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>

    <!--测试环境-->
    <profile>
        <id>dev</id>
        <properties>
            <profiles.activation>dev</profiles.activation>
        </properties>
    </profile>
</profiles>
    
```

**3 打包特定环境的资源**

```
<build>
	<finalName>winner-mvn-profile</finalName>
	<resources>
	    <resource>
	        <directory>src/main/resources/${profiles.activation}</directory>
	    </resource>
	</resources>
</build>


```

**4 spring加载特定环境属性文件的属性到容器**

```
<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	<property name="ignoreUnresolvablePlaceholders" value="true"/>
	<property name="locations">
	    <list>
	        <value>classpath:${profiles.activation}/config.properties</value>
	        <value>classpath:xxx.properties</value>
	    </list>
	</property>
</bean>

```

**5 动态指定环境**

```
mvn install -P local

```

