## 责任链模式

#### 定义

即设置一组处理器，对数据进行连续处理

#### 角色

* 处理器，例如MsgHandler， 对该角色扩展开放
* 处理器链， MsgHandlerChain，维护和迭代处理器，有两种方式：
	* 用数组保存所有处理器，链负责迭代调用
	* 仅保存处理器头结点，每个处理器都包含下一个处理器的引用，处理器自己进行迭代


#### 实现

**1 定义消息处理器**

```

/**
 * The {@code MsgHandler} interface providers
 * 消息处理器接口，链表节点
 *
 * 1 可处理消息
 * 2 可获取下一个处理器
 *
 * @author  liam
 * @version 1.0
 */
public interface MsgHandler {


    /**
     * 消息处理方法
     * @param msg 待处理的消息
     * @return 处理后的消息
     */
    public Object handle(Object msg);

    /**
     *  获取下一个handler
     * @return 下一个handler
     */
    public MsgHandler next();

    /**
     * 设置下一个handler
     * @param msgHandler 下一个handler
     */
    public void setNext(MsgHandler msgHandler);


}

```

**2 定义处理器责任链**

```

/**
 * The {@code MsgFilterChain}
 * mq消息过滤责任链，用链表实现
 *
 * @author  liam
 * @version 1.0
 */
public class MsgHandlerChain {

    //消息处理器列表
    private List<MsgHandler> msgHandlerList = new ArrayList<>();
    
    /**
     * 添加handler并形成handler链
     *
     * @param msgHandler 消息处理器
     * @return 处理器链本身
     */
    public MsgHandlerChain addHandler(MsgHandler msgHandler) {

        //形成handler链
        if (!msgHandlerList.isEmpty()) {
            msgHandlerList.get(msgHandlerList.size() - 1).setNext(msgHandler);
        }

        msgHandlerList.add(msgHandler);
        return this;
    }

    /**
     * 处理消息，调用处理器头结点开始处理
     * @param msg 待处理消息
     * @return 处理结果
     */
    public Object handle(Object msg) {
        return this.msgHandlerList.get(0).handle(msg);
    }

```

**3 定义处理器抽象类**

```
/**
 * The {@code AbstractMsgHandler}
 * 消息处理器抽象类，包含下一个处理器的引用
 * 消息具体处理交给子类实现
 *
 * @author  liam
 * @version 1.0
 */
public abstract class AbstractMsgHandler implements MsgHandler {

    //下一个handler
    private MsgHandler nextHandler;


    /**
     * 模板方法，实现处理器传递
     * @param msg 待处理的消息
     * @return 处理的结果，如果未空则终止处理
     */
    @Override
    public Object handle(Object msg) {

        Object result = doHandler(msg);
        if (result != null){
            if (nextHandler != null){
                return nextHandler.handle(result);
            }
        }
        return result;
    }

    /**
     *  获取下一个handler
     * @return 下一个handler
     */
    @Override
    public MsgHandler next(){
        return nextHandler;
    }

    /**
     * 设置下一个handler
     * @param msgHandler 下一个handler
     */
    @Override
    public void setNext(MsgHandler msgHandler){
        nextHandler = msgHandler;
    }

    /**
     * 模板方法，交给具体消息处理器实现
     * @param msg 待处理消息
     * @return 处理结果
     */
    public abstract Object doHandler(Object msg);
}

```

**4 定义处理器**

```

/**
 * The {@code MsgTimeoutHandler} class providers
 * 消息超时处理，过滤超时消息
 *
 * @author  liam
 * @version 1.0
 */
public class MsgTimeoutHandler extends AbstractMsgHandler{

    private static final Logger logger = LoggerFactory.getLogger(MsgUtil.class);

    //默认超时时间3min
    private static final long MSG_TIME_OUT = 3 * 60 * 1000;

    @Override
    public Object doHandler(Object msg) {

        //仅处理Map格式且包含时间戳的消息，否则交给下一个handler处理
        if (msg instanceof Map){
            Map mapMsg = (Map)msg;
            //如果消息包含时间戳，则进行超时处理
            if (mapMsg.get("timestamp") != null) {
                Long timeStamp = (Long) mapMsg.get("timestamp");
                if (System.currentTimeMillis() - timeStamp  - MSG_TIME_OUT > 0){
                    logger.warn("doHandler -- 该消息已超时 -- {}", msg);
                    return null;
                }
            }
        }

        //返回处理后的消息
        return msg;
    }

```

**5 调用处理器链进行消息处理**

```

//1 添加处理器
MsgHandlerChain msgHandlerChain.addHandler(new MsgFormateHandler()).addHandler(new MsgTimeoutHandler());

//2 开始处理
 Map<String, Object> mapMsg = (Map<String, Object>) msgHandlerChain.handle(currentMsg);
        
        
```