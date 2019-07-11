##观察者模式

#### 定义

观察者模式实际上也是发布订阅模式，本质上是一对多的消息广播


#### 角色
* Observer 观察者，对该角色扩展开放
* Observable 被观察者，对该角色修改关闭


#### 实践

使用java.util提供的观察者模式相关接口实现

* 定义被观察者

```
//继承Observable
public class UserSceneModifyHelper extends Observable implements InitializingBean
```

Observable维护观察者列表，在初始化时进行注入

```
/**
* 对象实例化后初始化观察者 TODO TEST
* @throws Exception
*/
@Override
public void afterPropertiesSet() throws Exception {
    this.addObserver(getGateWayUserSceneObserver());
}
```

* 定义观察者，可定义多个

```
//实现Observer接口
@Component
public class GateWayUserSceneObserver implements Observer


/**
 * 实现观察者的回调方法
 * 收到场景变更消息处理
 * @param o
 * @param arg
 */
@Override
public void update(Observable o, Object arg) {

    //TODO TEST
    Map<String, Object> notifyParam = new HashMap<>();
    Long userSceneId = (Long) notifyParam.get("userSceneId");
    Integer userId = (Integer) notifyParam.get("userId");
    notifyUserSceneUpdateMsg(userSceneId, userId);

}

```

* 消息通知

```
Map<String, Object> notifyParams = new HashMap<>();
notifyParams.put("userSceneId", userId);
notifyParams.put("userId", userId);
//被观察者接口,需要修改change flag
this.setChanged();
this.notifyObservers(notifyParams);        
```

```
public void notifyObservers(Object arg) {
    /*
     * a temporary array buffer, used as a snapshot of the state of
     * current Observers.
     */
    Object[] arrLocal;

	//同步获取观察者列表，线程安全
    synchronized (this) {
        /* We don't want the Observer doing callbacks into
         * arbitrary code while holding its own Monitor.
         * The code where we extract each Observable from
         * the Vector and store the state of the Observer
         * needs synchronization, but notifying observers
         * does not (should not).  The worst result of any
         * potential race-condition here is that:
         * 1) a newly-added Observer will miss a
         *   notification in progress
         * 2) a recently unregistered Observer will be
         *   wrongly notified when it doesn't care
         */
        if (!changed)
            return;
        arrLocal = obs.toArray();
        clearChanged();
    }

    //通知所有观察者
    for (int i = arrLocal.length-1; i>=0; i--)
        ((Observer)arrLocal[i]).update(this, arg);
}

```




