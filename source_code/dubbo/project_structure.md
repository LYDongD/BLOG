## 项目结构

#### 模块组织

> 核心模块关系

![核心关系](https://raw.githubusercontent.com/LYDongD/graphic/master/markdown/dubbo_modul01.png)

> maven pom组织关系

![pom](https://raw.githubusercontent.com/LYDongD/graphic/master/markdown/dubbo_modul02.png)

> 其他模块

![其他](https://raw.githubusercontent.com/LYDongD/graphic/master/markdown/dubbo_modul03.png)

---

#### 核心知识点

> maven bom 统一管理版本

**1 定义bom项目, 在bom项目的pom管理版本**

例如dubbo-dependencies-bom的pom.xml

```

<properties>
	<netty_version>3.2.5.Final</netty_version>
	<spring_version>4.3.16.RELEASE</spring_version>
</properties>

<dependencyManagement>
        <dependencies>
            <!-- 使用spring bom 统一依赖一个spring版本-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-framework-bom</artifactId>
                <version>${spring_version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            
            <dependency>
                <groupId>org.jboss.netty</groupId>
                <artifactId>netty</artifactId>
                <version>${netty_version}</version>
            </dependency>
<dependencyManagement>


```

**2 在parent的pom内导入bom**

```

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-dependencies-bom</artifactId>
                <version>2.7.0-SNAPSHOT</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

```

---

### 思考

* clife-parent采用什么方式统一管理依赖版本，是否采用了bom机制，例如导入spring bom？






