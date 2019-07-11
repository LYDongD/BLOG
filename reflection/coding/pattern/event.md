## 事件通知机制

> 角色

* 事件
* 事件监听器
* 事件发布管理器


[组件关系图参考](https://www.processon.com/view/link/5cdcc26de4b003096ddf2766)

---

> 事件

可继承事件实现字段扩展等

```
/**
 * 柜机事件
 * @author Liam
 * @date 2019/5/20 上午10:08
 */
public class CabinetEvent {

    /**
     *  事件来源
     */
    private final CabinetEventSourceEnum source;

    /**
     *  事件类型
     */
    private final CabinetEventTypeEnum type;

    /**
     *  事件发布的时间
     */
    private final Long timestamp;


    public CabinetEvent(CabinetEventSourceEnum source, CabinetEventTypeEnum type, Long timestamp) {

        if (type == null) {
            throw new CabinetEventException("事件类型不能为空");
        }

        if (source == null) {
            throw new CabinetEventException("事件源不能为空");
        }

        if (timestamp == null) {
            timestamp = System.currentTimeMillis();
        }

        this.source = source;
        this.type = type;
        this.timestamp = timestamp;
    }

    public Object getSource() {
        return source;
    }

    public CabinetEventTypeEnum getType() {
        return type;
    }

    public Long getTimestamp() {
        return timestamp;
    }

    @Override
    public String toString() {
        return "CabinetEvent{" +
                "source=" + source +
                ", type=" + type +
                ", timestamp=" + timestamp +
                '}';
    }
}

```

> 事件监听器

```
/**
 * 柜机事件监听器，实现该接口并声明为spring容器组件即可监听指定类型的柜机事件
 * @author Liam
 * @date 2019/5/20 上午10:17
 */
public interface CabinetEventListener<E extends CabinetEvent> {

    /**
     * 事件回调
     * @param cabinetEvent 柜机事件
     */
    void onCabinetEvent(E cabinetEvent);

    /**
     *  返回监听器支持的事件类型
     */
    CabinetEventTypeEnum supportEventType();
}

```

具体实现

```

/**
 * 柜机格口状态同步事件监听器
 * @author Liam
 * @date 2019/5/20 上午10:31
 */
@Slf4j
@Component
public class CabinetBoxSyncEventListener implements CabinetEventListener<CabinetEvent>{


    @Autowired
    private UnifyBoxDetailOperateFacade unifyBoxDetailOperateFacade;

    @Autowired
    private CabinetSyncDtoConverter cabinetSyncDtoConverter;

    /**
     *  柜机格口状态同步监听器仅处理柜机格口同步事件
     */
    private static final CabinetEventTypeEnum TYPE = CabinetEventTypeEnum.CABINET_BOX_SYNC_EVENT;

    @Override
    public void onCabinetEvent(CabinetEvent cabinetEvent) {
        if (!canSupport(cabinetEvent)) {
            log.error("柜机事件监听器不支持该事件类型 | event: {}", cabinetEvent.toString());
            return;
        }

        CabinetBoxSyncEvent cabinetBoxSyncEvent = (CabinetBoxSyncEvent) cabinetEvent;

        //模型转换
        OperateCellBusinessPostReqDto operateCellBusinessPostReqDto = cabinetSyncDtoConverter.convert(cabinetBoxSyncEvent.getCabinetBoxSyncData());
        if (operateCellBusinessPostReqDto == null) {
            log.error("柜机格口同步，模型转换失败");
            throw new CabinetEventException("柜机格口同步，模型转换失败");
        }

        ResultDto resultDto = unifyBoxDetailOperateFacade.postHandle(operateCellBusinessPostReqDto);
        if (!resultDto.getSuccess()) {
            log.error("格口状态更新RPC调用失败 | {} ", resultDto.getRespMsg());
            throw new CabinetEventException("格口状态更新RPC调用失败 | " + resultDto.getRespMsg());
        }

        log.info("格口状态更新RPC调用成功 " + JSON.toJSONString(operateCellBusinessPostReqDto));
    }

    /**
     * 能否支持该柜机事件
     * @param cabinetEvent 柜机事件
     * @return 是否支持
     */
    private Boolean canSupport(CabinetEvent cabinetEvent) {
        if (cabinetEvent == null || !TYPE.equals(cabinetEvent.getType())) {
            return false;
        }
        return cabinetEvent instanceof CabinetBoxSyncEvent;
    }

    @Override
    public CabinetEventTypeEnum supportEventType() {
        return TYPE;
    }
}


```

> 事件发布管理器

* 管理事件监听器路由表
* 发布事件，支持同步/异步

```

/**
 * 柜机事件发布者，发布一个柜机事件
 * @author Liam
 * @date 2019/5/20 上午10:04
 */
@Slf4j
@Component
public class CabinetEventPublisher {


    /**
     *  是否同步发布事件，默认异步
     */
    private Boolean syncPublish;

    /**
     *  柜机事件发布专用线程池
     */
    @Autowired
    private CabinetEventPublishExecutor executor;

    @Autowired
    private List<CabinetEventListener<CabinetEvent>> cabinetEventListeners;

    /**
     *  柜机事件监听器路由表，根据事件类型进行路由
     */
    private final Map<CabinetEventTypeEnum, List<CabinetEventListener<CabinetEvent>>> cabinetEventListenersMap = new ConcurrentHashMap<>();


    /**
     *  初始化路由表, 注册监听器
     *  从容器中获取所有的柜机事件监听器，并建立映射关系
     */
    @PostConstruct
    public void registerListeners(){
        if (!CollectionUtils.isEmpty(cabinetEventListeners)) {
            for (CabinetEventListener<CabinetEvent> cabinetEventListener : cabinetEventListeners) {
                if (cabinetEventListenersMap.get(cabinetEventListener.supportEventType()) == null) {
                    List<CabinetEventListener<CabinetEvent>> cabinetEventListenerList = new LinkedList<>();
                    cabinetEventListenerList.add(cabinetEventListener);
                    cabinetEventListenersMap.put(cabinetEventListener.supportEventType(), cabinetEventListenerList);
                }else {
                    cabinetEventListenersMap.get(cabinetEventListener.supportEventType()).add(cabinetEventListener);
                }
            }
        }

        log.info("柜机事件发布管理器初始化完毕 | 路由表：{} ", cabinetEventListenersMap);
    }


    /**
     * 发布柜机事件
     * @param cabinetEvent 柜机事件
     */
    public void publishEvent(CabinetEvent cabinetEvent){
        List<CabinetEventListener<CabinetEvent>> cabinetEventListeners = cabinetEventListenersMap.get(cabinetEvent.getType());
        if (CollectionUtils.isEmpty(cabinetEventListeners)) {
            log.warn("没有找到该事件的监听器 | event: {}", cabinetEvent.toString());
            return;
        }

        //异步发布事件
        for (CabinetEventListener<CabinetEvent> cabinetEventListener : cabinetEventListeners) {
            log.info("发布柜机事件 | event: {}", cabinetEvent.toString());
            if (syncPublish == null || !syncPublish) { //异步
                executor.execute(() -> {
                    invokeListener(cabinetEventListener, cabinetEvent);

                });
            }else { //同步
                invokeListener(cabinetEventListener, cabinetEvent);
            }
        }
    }


    /**
     * 调用监听器的事件监听方法
     * @param cabinetEventListener 柜机事件监听器
     * @param cabinetEvent 柜机事件
     */
    private void invokeListener(CabinetEventListener<CabinetEvent> cabinetEventListener, CabinetEvent cabinetEvent) {
        try {
            cabinetEventListener.onCabinetEvent(cabinetEvent);
            log.info("柜机事件发布成功 | event: {}", cabinetEvent.toString());
        }catch (Exception e){
            //暂不处理事件发布失败的情况
            log.error("柜机事件发布失败 | event: {}", cabinetEvent.toString(), e);
        }
    }
}


```


