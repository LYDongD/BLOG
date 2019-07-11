## maven

### 继承与聚合

> 如何将springboot的parent继承改成依赖

参考[springboot去parent](https://www.baeldung.com/spring-boot-dependency-management-custom-parent)

即只需要将parent的依赖管理和插件添加到当前模块即可

### 常用插件

> 打包发布

```

#构建时跳过测试
-Dmaven.test.skip=true

#设置模块及子模块的统一版本
mvn versions:set -DnewVersions=4.1.71

#发布到maven仓库
mvn deploy

#检查依赖版本是否为正式版
#检查文件是否提交
#检出tag
#构建并发布到maven仓库
#更新当前模块版本为snapshot
mvn release:prepare -Dresume=false
mvn release:perform -DuseReleaseProfile=false



```

> 自动生成mybatis映射文件

```
 <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.CabinetBaseServerApplication</mainClass>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <configuration>
                    <!--<configurationFile>src/main/resources/mybatis-generator/generatorConfig.xml</configurationFile>-->
                    <!--<configurationFile>src/main/resources/mybatis-generator/generatorConfig_heartConfig.xml</configurationFile>-->
                    <configurationFile>src/main/resources/mybatis-generator/generatorConfigMysql_cabinetRouter.xml</configurationFile>
                    <verbose>true</verbose>
                    <overwrite>true</overwrite>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>com.oracle</groupId>
                        <artifactId>ojdbc14</artifactId>
                        <version>${version.oracle}</version>
                        <scope>runtime</scope>
                    </dependency>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>${version.mysql}</version>
                        <scope>runtime</scope>
                    </dependency>
                </dependencies>
            </plugin>

```

对应的generator配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
 <!DOCTYPE generatorConfiguration
         PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
         "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
 
 <generatorConfiguration>
     <context id="mysqlGenerator" targetRuntime="MyBatis3">
         <plugin type="org.mybatis.generator.plugins.CaseInsensitiveLikePlugin" />
         <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />

         <commentGenerator>
             <property name="suppressDate" value="true"/>
             <property name="suppressAllComments" value="true"/>
         </commentGenerator>

         <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                         connectionURL="jdbc:mysql://xxx:3306/demo_locker?useUnicode=true&amp;characterEncoding=utf-8" userId="xxxx"
                         password="xxxx"/>

         <javaTypeResolver >
             <property name="forceBigDecimals" value="false" />
         </javaTypeResolver>

         <javaModelGenerator targetPackage="com.demo.base.server.model.mysql"
                             targetProject="src/main/java">
             <property name="enableSubPackages" value="false"/>
             <property name="trimStrings" value="false"/>
         </javaModelGenerator>

         <sqlMapGenerator targetPackage="mysqlMappers"
                          targetProject="src/main/resources">
             <property name="enableSubPackages" value="false"/>
         </sqlMapGenerator>

         <javaClientGenerator targetPackage="com.demo.base.server.dao.mysql"
                              targetProject="src/main/java" type="XMLMAPPER">
             <property name="enableSubPackages" value="false"/>
         </javaClientGenerator>

         <!---->
          <table tableName="demo_cabinet_router_base"
                 enableInsert="true" enableCountByExample="true"
                 enableSelectByPrimaryKey="true" enableSelectByExample="true"
                 enableUpdateByPrimaryKey="true" enableUpdateByExample="true"
                 enableDeleteByPrimaryKey="true" enableDeleteByExample="true">
              <generatedKey column="ID" sqlStatement="MySql" identity="true" />
              <columnOverride column="ID" jdbcType="INTEGER" javaType="java.lang.Long" />
              <columnOverride column="network_operator" jdbcType="TINYINT" javaType="java.lang.Integer" />
          </table>

     </context>
 </generatorConfiguration>

```

执行插件

[参考](http://www.mybatis.org/generator/running/runningWithMaven.html)

```
mvn mybatis-generator:generate

```

> 依赖管理

```
#强制更新snapshot版本
mvn clean install -U

```


### 环境配置

```
#构建时指定profiles=site1
mvn clean install -Dmaven.test.skip=true -Psite1

```

对应pom

```
<profiles>
        <profile>
            <id>local</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <spring.profiles.active>local</spring.profiles.active>
            </properties>
        </profile>
        <profile>
            <id>dev</id>
            <!--<activation>-->
                <!--<activeByDefault>true</activeByDefault>-->
            <!--</activation>-->
            <properties>
                <spring.profiles.active>dev</spring.profiles.active>
            </properties>
        </profile>
        <profile>
            <id>site1</id>
            <properties>
                <spring.profiles.active>site1</spring.profiles.active>
            </properties>
        </profile>
</profiles>

```

#### 功能配置

> 使用filter机制实现配置文件内的变量替换

resource: 将资源文件打包到classpath下, 默认情况, maven会从src/main/resources查找资源，如果不在这里，才需要特别指定

* 资源文件可定义变量，通过${}的方式或@xxx@的方式(springboot)
* 构建mvn项目时，相关变量会根据filter指定的文件或其他方式替换成具体值

利用build-resource 的filter机制以及插件使用了插件maven-resources-plugin

```
<resources>
    <resource>
        <directory>src/main/resources</directory>
        //排除不需要打包进来的文件
        <excludes>
            <exclude>config/disconf/*.*</exclude>
            <exclude>config/local/*.*</exclude>
            <exclude>mybatis-generator/*.*</exclude>
            <exclude>assembly/*.*</exclude>
        </excludes>
        //开启filter，可替换变量
        <filtering>true</filtering>
    </resource>
</resources>

//指定属性文件内的值作为变量值
<filters>
    <filter>src/main/resources/assembly/local.properties</filter>
</filters>

```

参考 <https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html>


