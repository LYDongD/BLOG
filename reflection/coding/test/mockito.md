## 使用mockito进行单元测试

### springboot快速接入

```

//测试类必须添加runner,通过runner初始化上下文，并创建测试类进行测试
@RunWith(SpringRunner.class)
@SpringBootTest
public class BaseTest {

}

```

### spring快速接入

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:spring/applicationContext.xml"})
public class BaseAppTest {

    //初始化mockito注解：扫描测试类的mockito注解并解析
    //解析注解，即调用Mockito.mock创建mock对象并注入
    @Before
    public void initMockito() {
        MockitoAnnotations.initMocks(this);
    }
}

```

### 去spring快速接入

对于上诉两种方式，每次测试都需要启动spring容器，创建引入大量不必要的对象，耗时较大

使用@InjectMock 和 @Mock 进行测试，不引入spring上下文

```
public class BoxUsedDetailServiceImplTest extends BaseTest{

     @InjectMocks
     BoxUsedDetailServiceImpl boxUsedDetailService = new BoxUsedDetailServiceImpl();

     @Mock
     BoxCmdMapper boxCmdMapper;
     @Mock
     BoxUsedMapper boxUsedMapper;
     @Mock
     BoxExceptionService boxExceptionService;
     @Mock
     CabinetBoxSyncHandler cabinetBoxSyncHandler;

     @Test
     public void operateWithNoException() {
        //正常格口更新
        LockerOperateRequest lockerOperateRequest = new LockerOperateRequest();
        lockerOperateRequest.setBusyException(BusyExceptionEnums.ExceptionTypeEnum.NOMAL.type);
        lockerOperateRequest.setEdCode("FC7552707");
        lockerOperateRequest.setBoxId("R0102");
        lockerOperateRequest.setBusinessType(BoxUsedDetailEnums.BoxBizType.STORAGE.type);
        when(boxUsedMapper.updateBoxUsedDetailByDto(any(EdmBoxCmdUsedDetail.class))).thenReturn(1);
        when(boxCmdMapper.insertEdmBoxException(any(EdmBoxException.class))).thenReturn(1);
        when(boxExceptionService.sendSmsAysn(any())).thenReturn(true);
        when(cabinetBoxSyncHandler.cabinetBoxSync(any(CabinetBoxSyncDto.class))).thenReturn(true);
        boolean result = boxUsedDetailService.operate(new LockerOperateRequest());
        //触发了格口同步，不发送短信(只有未开门才发送短信)
        verify(cabinetBoxSyncHandler, times(1)).cabinetBoxSync(any(CabinetBoxSyncDto.class));
        verify(boxExceptionService, times(0)).sendSmsAysn(any());
        Assert.assertTrue(result);
    }
}

```

### 基本使用

1 使用@mock，模拟对象的所有行为

* 使用when(xxx).thenReturn(xxx)实现stubbing
* 使用ReflectionTestUtils实现mock注入到spring容器bean

```
@Slf4j
public class Fault2OperateFacadeImplTest extends BaseTest {


    @Resource(name = "fault2OperateFacade")
    Fault2OperateFacade fault2OperateFacade;

    @Mock
    Dispatcher dispatcher;


    /**
     * 故障统一上报接口
     * 1 故障版本为1
     * 2 故障faultId不为空
     */
    @Test
    public void record() {
        FaultRecordReqDto faultRecordReqDto = new FaultRecordReqDto();
        when(dispatcher.checkForRecordAndPushKafka(faultRecordReqDto)).thenReturn(CommonUtils.assembleResultDto(ResponseEnum.SUCCESS));
        ReflectionTestUtils.setField(fault2OperateFacade, "dispatcher", dispatcher);
        ResultDto resultDto = fault2OperateFacade.record(faultRecordReqDto);
        Assert.assertEquals(true, resultDto.getSuccess());
        Assert.assertNotNull(faultRecordReqDto.getFaultId());
        Assert.assertEquals(1, faultRecordReqDto.getFaultVersion().intValue());
    }
}
```

2 使用@spy，模拟对象的特定行为

* 使用doReturn(xxx).when(xxx) 实现特定方法的stubbing，其他方法不会被mock

```
public class DispatcherTest extends BaseTest {


    @Spy
    Dispatcher dispatcher;

    @Mock
    KafkaApiProducerService kafkaApiProducerService;

    @Mock
    CellFaultService cellFaultService;

    /**
     *  校验参数，转换模型，并推送至kafka
     *  验证方法调用流程
     */
    @Test
    public void checkForRecordAndPushKafka() {

        Mockito.doNothing().when(kafkaApiProducerService).pushCabinetFaultInfoToHandleQueue(any(CabinetFaultInfoDto.class));
        ReflectionTestUtils.setField(dispatcher, "kafkaApiProducerService", kafkaApiProducerService);


        ResultDto<CabinetFaultInfoDto> cabinetFaultInfoDtoResultDto = new ResultDto<>();
        cabinetFaultInfoDtoResultDto.setData(new CabinetFaultInfoDto());
        cabinetFaultInfoDtoResultDto.setSuccess(true);
        cabinetFaultInfoDtoResultDto.setRespCode(ResponseEnum.SUCCESS.respCode);
        doReturn(cabinetFaultInfoDtoResultDto).when(dispatcher).checkForRecord(any(FaultRecordReqDto.class));

        FaultRecordReqDto faultRecordReqDto = new FaultRecordReqDto();
        ResultDto resultDto = dispatcher.checkForRecordAndPushKafka(faultRecordReqDto);

        //验证该方法会进行参数校验并推送kafka
        verify(dispatcher, times(1)).checkForRecord(any(FaultRecordReqDto.class));
        verify(kafkaApiProducerService, times(1)).pushCabinetFaultInfoToHandleQueue(any(CabinetFaultInfoDto.class));
        Assert.assertTrue(resultDto.getSuccess());
    }

```


3 使用verify跟踪mock对象的行为

* 跟踪mock对象调用的次数
* 跟踪mock对象调用的顺序

```
//验证该方法会进行参数校验并推送kafka
verify(dispatcher, times(1)).checkForRecord(any(FaultRecordReqDto.class));
verify(kafkaApiProducerService, times(1)).pushCabinetFaultInfoToHandleQueue(any(CabinetFaultInfoDto.class));

```

4 参数匹配器

* 使用Match进行策略匹配，支持类型匹配，任意匹配等

```
//匹配任意FaultRecordReqDto类型的对象
verify(dispatcher, times(1)).checkForRecord(any(FaultRecordReqDto.class));
```

5 静态方法和私有方法 mock

使用PowerMock

> 引入powermock

注意版本匹配

```
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-all</artifactId>
            <version>1.10.19</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-api-mockito</artifactId>
            <version>1.6.4</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-module-junit4</artifactId>
            <version>1.6.4</version>
            <scope>test</scope>
        </dependency>

```

版本对照如下：

```
Mockito                     | PowerMock
------------------------------------------------------------------------------
2.0.0-beta - 2.0.42-beta    |   1.6.5+
------------------------------------------------------------------------------
1.10.19                     |   1.6.4
1.10.8 - 1.10.x             |   1.6.2+
1.9.5-rc1 - 1.9.5           |   1.5.0 - 1.5.6
1.9.0-rc1 & 1.9.0           |   1.4.10 - 1.4.12
1.8.5                       |   1.3.9 - 1.4.9
1.8.4                       |   1.3.7 & 1.3.8
1.8.3                       |   1.3.6
1.8.1 & 1.8.2               |   1.3.5
1.8                         |   1.3
1.7                         |   1.2.5

```

> 启用powermock上下文

指定需要测试的私有方法所在类 @PrepareForTest

```
@RunWith(PowerMockRunner.class)
@PrepareForTest(BoxUsedDetailServiceImpl.class)
public class BoxUsedDetailServiceImplTest extends BaseTest{

    /**
     *  暂存业务，格口占用字段映射
     */
    @Test
    public void convertBoxUsedDetailByOccupy() throws Exception{

        LockerOperateRequest lockerOperateRequest = new LockerOperateRequest();
        lockerOperateRequest.setEdCode("FC7552707");
        lockerOperateRequest.setBoxId("R0102");
        lockerOperateRequest.setBusinessType(BoxUsedDetailEnums.BoxBizType.STORAGE.type);
        lockerOperateRequest.setOperateType(LockerOperateEnum.OCCUPY.type);
        lockerOperateRequest.setBusyException(BusyExceptionEnums.ExceptionTypeEnum.NOMAL.type);
        lockerOperateRequest.setDdsId("testDsdid");

        //测试私有方法
        Method method = PowerMockito.method(BoxUsedDetailServiceImpl.class, "convertBoxUsedDetailByOccupy", LockerOperateRequest.class);
        EdmBoxCmdUsedDetail edmBoxCmdUsedDetail = (EdmBoxCmdUsedDetail)method.invoke(boxUsedDetailService, lockerOperateRequest);

        //关注核心字段
        Assert.assertEquals(BoxUsedDetail.OCCUPIED.getStatus(), edmBoxCmdUsedDetail.getBoxCondition());
        Assert.assertEquals(BoxUsedDetailEnums.BoxBizType.STORAGE.type, edmBoxCmdUsedDetail.getBusinessType());
        Assert.assertEquals(BoxFaultEnum.BOX_FAULT_ENUM_FALSE.getCode(), edmBoxCmdUsedDetail.getIsFault());

        //暂存id不为空
        Assert.assertTrue(!edmBoxCmdUsedDetail.getDdsId().isEmpty());
        Assert.assertTrue(edmBoxCmdUsedDetail.getPostid().isEmpty());
        Assert.assertTrue(edmBoxCmdUsedDetail.getOrderid().isEmpty());
        Assert.assertTrue(edmBoxCmdUsedDetail.getSampleOrderId().isEmpty());
    }
}

```

5 结合@spy, @InjectMock和Powermock

实现目标测试对象的跟踪，stubbing私有方法和依赖对象

```
@InjectMocks
@Spy
CellDaoServiceImpl cellDaoService = new CellDaoServiceImpl();

@Mock
FcboxSpecialBoxSettingMapper fcboxSpecialBoxSettingMapper;



    /**
     * 专用格口释放
     */
    @Test
    public void specialBoxSettingRelease() throws Exception{

        //mock private method
        PowerMockito.doReturn(true).when(cellDaoService, "speicalBoxSettingInsertSuccess", any(FcboxSpecialBoxSetting.class));

        doReturn(1).when(fcboxSpecialBoxSettingMapper).updateByExampleSelective(any(FcboxSpecialBoxSetting.class), any(FcboxSpecialBoxSettingExample.class));
        doReturn(1).when(fcboxSpecialBoxSettingMapper).insert(any(FcboxSpecialBoxSetting.class));
        doReturn(1).when(edmBoxCmdUsedDetailMapper).updateByExampleSelective(any(EdmBoxCmdUsedDetail.class), any(EdmBoxCmdUsedDetailExample.class));


        SpecialBoxDto specialBoxDto = new SpecialBoxDto();
        specialBoxDto.setCode(CABINET_CODE);
        specialBoxDto.setBoxId(BOX_ID);
        specialBoxDto.setOperator(OPERATOR);
        specialBoxDto.setSpecialCode(0);
        String businessId = cellDaoService.specialBoxSetting(specialBoxDto);

        verify(fcboxSpecialBoxSettingMapper, times(1)).updateByExampleSelective(any(FcboxSpecialBoxSetting.class), any(FcboxSpecialBoxSettingExample.class));
        verify(fcboxSpecialBoxSettingMapper, times(0)).insert(any(FcboxSpecialBoxSetting.class)); //格口释放会插入新的格口
        verify(edmBoxCmdUsedDetailMapper, times(1)).updateByExampleSelective(any(EdmBoxCmdUsedDetail.class), any(EdmBoxCmdUsedDetailExample.class));
        Assert.assertNotNull(businessId);
        Assert.assertEquals(0, businessId.length()); //是否格口，业务id被清空
    }

```
