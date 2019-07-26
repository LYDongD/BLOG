## spring + mybatis 多数据源配置

### XML配置

datasource.xml, 配置数据源1，oracle

```
	<!-- druid数据源  -->
	<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close" lazy-init="true">
		<property name="driverClassName" value="${jdbc.driver}" />
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
	</bean>

    <!-- mybatis文件配置，扫描所有mapper文件 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations" value="classpath*:mappers/*Mapper.xml" />
        <property name="typeAliasesPackage" value="com.liam.demo.resourcepool.fault.model" />
    </bean>


    <bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
        <property name="basePackage" value="com.liam.demo.resourcepool.fault.dao.oracle" />
    </bean>

```

datasource-mysql.xml， 配置数据源2， mysql

```
	<!-- druid数据源  -->
	<bean id="dataSourceMysql" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close" lazy-init="true">
		<property name="driverClassName" value="${mysql.jdbc.driver}" />
		<property name="url" value="${mysql.jdbc.url}" />
		<property name="username" value="${mysql.jdbc.username}" />
		<property name="password" value="${mysql.jdbc.password}" />
	</bean>


    <!-- mybatis文件配置，扫描所有mapper文件 -->
    <bean id="sqlSessionFactoryMysql" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSourceMysql" />
        <property name="mapperLocations" value="classpath*:mysqlmappers/*Mapper.xml" />
        <property name="typeAliasesPackage" value="com.liam.demo.resourcepool.fault.model, com.liam.demo.resourcepool.common.cache.model,
        com.liam.demo.resourcepool.cell.model, com.liam.demo.resourcepool.cabinet.model" />
    </bean>

    <bean id="mapperScannerConfigurerMysql" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryMysql" />
        <property name="basePackage" value="com.liam.demo.resourcepool.fault.dao.mysql, com.liam.demo.resourcepool.common.cache.dao,
        com.liam.demo.resourcepool.cell.dao, com.liam.demo.resourcepool.cabinet.dao" />
    </bean>

    <!-- 事务注解配置 -->
    <tx:annotation-driven transaction-manager="txManagerMysql" mode="proxy" />

    <!-- 对数据源进行事务管理 -->
    <bean id="txManagerMysql"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSourceMysql" />
    </bean>

    <!-- 提供手动控制事物模板 -->
    <bean id="transactionTemplateMysql"
          class="org.springframework.transaction.support.TransactionTemplate">
        <property name="transactionManager">
            <ref bean="txManagerMysql" />
        </property>
        <!--ISOLATION_DEFAULT 表示由使用的数据库决定 -->
        <property name="isolationLevelName" value="ISOLATION_DEFAULT" />
        <property name="propagationBehaviorName" value="PROPAGATION_REQUIRED" />
        <property name="timeout" value="30" />
    </bean>
</beans>

```

applicationContext.xml 导入以上两个数据源配置

```
<import resource="datasource.xml" />
<import resource="datasource-mysql.xml" />

```

容器启动时，加载applicationContext配置文件

配置类的方式实现：

```
@Configuration
@ImportResource(locations={"classpath:spring/applicaitonContext.xml"})
public class ConfigClass {
    
}

```

### springboot的配置方式

> 依赖

```
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid.version}</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${version.mybatis}</version>
        </dependency>

        <!-- 使用oracle数据库驱动 -->
        <dependency>
            <groupId>com.oracle</groupId>
            <artifactId>ojdbc14</artifactId>
            <version>${version.oracle}</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${version.mysql}</version>
        </dependency>

```

> 通过Disconf配置中心拉取多数据源配置属性

application.yml

```
disconf:
  scanPackage: com.liam.config.domain
  conf_server_host: xxxx
  version: "1_0_0_0"
  files: multi-db.properties
  app: cabinet-base-server
  env: rd
  debug: false
  enable:
    remote:
      conf: true
    system:
      property: true
  conf_server_url_retry_times: 3
  conf_server_url_retry_sleep_seconds: 5
  user_define_download_dir: /app/spring-boot/disconf/cabinet-base-server
  enable_local_download_dir_in_class_path: false

```

multi-db.properties

```
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.druid.initial-size=32
spring.datasource.druid.max-active=64
spring.datasource.druid.min-idle=32
spring.datasource.druid.max-wait=2000
spring.datasource.druid.pool-prepared-statements=true
spring.datasource.druid.max-pool-prepared-statement-per-connection-size=5
spring.datasource.druid.validation-query=select 1 from dual
spring.datasource.druid.validation-query-timeout=1
spring.datasource.druid.test-on-borrow=false
spring.datasource.druid.test-on-return=false
spring.datasource.druid.test-while-idle=true
spring.datasource.druid.time-between-eviction-runs-millis=10000
spring.datasource.druid.min-evictable-idle-time-millis=30001
spring.datasource.druid.async-close-connection-enable=true
spring.datasource.druid.filters=stat
spring.datasource.druid.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=2000

#数据源1
spring.datasource.druid.oracle.driver-class-name=oracle.jdbc.driver.OracleDriver
spring.datasource.druid.oracle.url=jdbc:oracle:thin:@xxxx:151:xxx
spring.datasource.druid.oracle.username=xxx
spring.datasource.druid.oracle.password=xxx

#数据源2
spring.datasource.druid.mysql.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.druid.mysql.url=jdbc:mysql://xxxx:3306/liam_locker?useSSL=false&useUnicode=true&characterEncoding=utf-8
spring.datasource.druid.mysql.username=xxx
spring.datasource.druid.mysql.password=xxx

```

mysql数据源配置

```
@Configuration
@MapperScan(basePackages = {"com.liam.base.server.dao.mysql"},sqlSessionTemplateRef = "mysqlSqlSessionTemplate")
public class MysqlDatasourceConfig {

    @Bean(name = "mysqlDatasource")
    @ConfigurationProperties(prefix="spring.datasource.druid.mysql")
    public DataSource dataSource() {
        return DataSourceBuilder.create().type(com.alibaba.druid.pool.DruidDataSource.class).build();
    }

    @Bean(name = "mysqlSqlSessionFactory")
    public SqlSessionFactory setSqlSessionFactory(@Qualifier("mysqlDatasource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mysqlMappers/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "mysqlTransactionManager")
    public PlatformTransactionManager platformTransactionManager(@Qualifier("mysqlDatasource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "mysqlSqlSessionTemplate")
    @Primary
    public SqlSessionTemplate sqlSessionTemplate(@Qualifier("mysqlSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
    
```

oracle数据源配置

```
@Configuration
@MapperScan(basePackages = {"com.liam.base.server.dao.oracle", "com.liam.demo.terminal.dao", "com.sf.module.demo.box.mapper", "com.sf.demo.boxinfo.dao"},sqlSessionTemplateRef = "oracleSessionTemplate")
public class OracleDatasourceConfig {

    @Bean(name = "oracleDatasource")
    @ConfigurationProperties(prefix="spring.datasource.druid.oracle")
    public DataSource dataSource() {
        return DataSourceBuilder.create().type(com.alibaba.druid.pool.DruidDataSource.class).build();
    }

    @Bean(name = "oracleSqlSessionFactory")
    public SqlSessionFactory setSqlSessionFactory(@Qualifier("oracleDatasource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:liam-mappers/*.xml,classpath:mappers/*.xml,classpath:mapper/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "oracleTransactionManager")
    public PlatformTransactionManager platformTransactionManager(@Qualifier("oracleDatasource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "oracleSessionTemplate")
    @Primary
    public SqlSessionTemplate sqlSessionTemplate(@Qualifier("oracleSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}

```
