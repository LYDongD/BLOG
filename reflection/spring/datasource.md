## 数据源配置

### 通过JNDI引用外部数据源

配置JndiObjectFactoryBean

```
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
	<property name="jndiName">
		<value>java:jdbc/xxxxDataSource</value>
	</property>
</bean>

```

数据源由外部容器，例如JBOSS负责创建，并绑定到JNDI山下文中

JBOSS数据源配置： oracle.xml

```
<datasources>
  <local-tx-datasource>
    <jndi-name>jdbc/liamDataSource</jndi-name>
    <connection-url>jdbc:oracle:thin:@ (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = xxxx)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = xxxx)
      (INSTANCE_NAME = xxxx)
    ))</connection-url>
    <driver-class>oracle.jdbc.driver.OracleDriver</driver-class>
    <user-name>xxx</user-name>
    <password>xxx</password>
    <exception-sorter-class-name>org.jboss.resource.adapter.jdbc.vendor.OracleExceptionSorter</exception-sorter-class-name>
    <min-pool-size>50</min-pool-size>
    <max-pool-size>100</max-pool-size>
    <blocking-timeout-millis>5000</blocking-timeout-millis>
    <idle-timeout-minutes>5</idle-timeout-minutes>
    <metadata>
      <type-mapping>Oracle10g</type-mapping>
    </metadata>
  </local-tx-datasource>
</datasources>

```

JNDI通过JNDI上下文(InitContext)查找关联数据源,并获取数据库连接

java api:

```
Context ctx=new InitialContext(); 
Object datasourceRef=ctx.lookup("jdbc/liamDataSource"); //在上下文中查找数据源，前提是JNDI服务提供者已经为上下文绑定该数据源
DataSource ds=(Datasource)datasourceRef; 
Connection conn=ds.getConnection();

```

数据源通过JNDI的方式进行配置，使应用无需关心数据源的具体实现，例如驱动和相关属性，只需要替换JNDI配置文件即可
