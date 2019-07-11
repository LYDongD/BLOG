## 数据库测试

### 自定义数据源连接池进行测试

#### 为什么要自定义连接池？

解决以下下问题：

1 应用采用外部数据源配置，测试用例无法快速和数据库建立连接

2 测试类想指定某个特定的数据源

3 为了避免污染数据库，需要引入事务，并精准控制事务


#### 解决方案

1 配置mybatis连接池： mybatis-settings.xml，指定数据源，别名, 日志框架和测试mapper

2 测试类加载该配置文件， 并初始化连接

3 使用mybatis的java api完成数据库操作

#### mybatis的全局配置

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

    <!--输出到控制台-->
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <!--别名-->
    <typeAliases>
        <package name="com.liam.module.xxx.domain"/>
    </typeAliases>

    <!--配置数据库参数，这里采用sit1-->
    <environments default="development">
        <environment id="development">
            <!--启用事务，默认autoCommit=false，需要显示提交或回滚-->
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="oracle.jdbc.driver.OracleDriver"/>
                <property name="url" value="jdbc:oracle:thin:@xxx:1521:xxx"/>
                <property name="username" value="xxx"/>
                <property name="password" value="xxx"/>
            </dataSource>
        </environment>
    </environments>


    <!--配置需要测试的mapper文件-->
    <mappers>
        <mapper resource="mappers/ManagerOperate_Mapper.xml"/>
    </mappers>

</configuration>

```

#### 测试基类

BaseDaoTest

```

/**
 * dao测试基类，用mybatis连接池作为数据源
 * 1 支持事务
 * 2 支持日志输出到stdout
 * 3 支持仅添加测试mapper
 * @author Liam(003046)
 * @date 2019/6/11 下午6:19
 */
@Slf4j
public class BaseDaoTest {

    protected SqlSessionFactory sqlSessionFactory;

    @Before
    public void initConnect() throws Exception{
        InputStream inputStream = Resources.getResourceAsStream("mybatis-settings.xml");
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }

}

```

#### 测试dao类

ManagerOperateTest， 继承BaseDaoTest获取会话仓库

```
/**
 * MapperOperate 数据库测试
 * @author Liam(003046)
 * @date 2019/6/28 下午5:25
 */

public class ManagerOperateTest extends BaseDaoTest {


    @Test
    public void updateBoxUsedDetailStatus() {

        SqlSession sqlSession = sqlSessionFactory.openSession();
        ManagerOperate managerOperate = sqlSession.getMapper(ManagerOperate.class);
        Map<String, String> boxUsedDetailMap = new HashMap<>();
        boxUsedDetailMap.put("status", "2");
        boxUsedDetailMap.put("assetId", "FC7001157");
        boxUsedDetailMap.put("boxId", "L0101");
        boxUsedDetailMap.put("isFault", "1");
        managerOperate.updateBoxUsedDetailStatus(boxUsedDetailMap);
        sqlSession.commit();
    }
}

```
