### LBS需求对接文档

#### 架构图

![LBS架构图](http://omdlmhd54.bkt.clouddn.com/lbs%E6%9E%B6%E6%9E%84.png)


#### LBS规则配置

1. 地点圈相关接口

	* 服务：CircleServiceFacade
	* 方法：

	
	```
	/**
     * 添加地点圈circle并返回主键
     * @param  circle 参数circle
     * @return 新插入记录主键
     * @throws
     */
	Long insertWithIdReturned(Circle circle);
	
	
	/**
     * 修改地点圈
     * @param circle 修改的数据
     * @return 修改的数据条数
     */
    public void update(Circle circle);
    
    
	/**
     * 删除地点圈
     * @param circleId 数据标识
     * @return 删除的数据条数
     */
    public void delete(Long circleId);
    
     /**
     * 通过id查询数据
     * @param circleId 数据标识
     * @return 数据对象
     */
    public T get(Long circleId);
     
	 /**
     * 通过条件查询地点圈列表
     * @param circle t 查询条件
     * @return 数据列表
     */    
    public List<T> getList(Circle circle);
    
	
	```

2. 用户条件相关配置接口

	* 服务：ConditionInstanceServiceFacade
	* 方法：
	
	```
	    /**
     * 批量新增或修改用户场景的条件（限同一个条件组）
     * @param userConditionInstanceList 用户条件实例列表
     * @throws com.clife.commons.base.exception.expert.ExpertServiceException 异常
     */
    void insertOrUpdateBatch(List<UserConditionInstance> userConditionInstanceList);


    /**
     * 添加单个用户条件实例（仅支持设备类条件）
     * @param userConditionInstance 用户条件实例
     * @return 条件实例id
     * @throws com.clife.commons.base.exception.expert.ExpertServiceException 异常
     */
    Long insertOrUpdate(UserConditionInstance userConditionInstance);


    /**
     * 批量删除条件实例
     * @param userConditionInstanceIdList  待删除的条件id数组
     * @throws ExpertServiceException
     */
    void deleteBatch(List<Long> userConditionInstanceIdList) throws ExpertServiceException;


    /**
     * 获取条件选项列表
     * @param userSceneId 用户场景ID
     * @return 条件选项列表
     */
    List<ConditionType> getConditionOptionList(Long userSceneId);


    /**
     * 设置规则条件关系
     * @param userSceneId 用户场景ID
     * @param subSceneIndex 规则序号
     * @param activateExpression 条件组的逻辑关系符
     */
     void setCondtitionRelation(Long userSceneId, Integer subSceneIndex, String activateExpression);


    /**
     * 批量更新场景条件关系
     * @param userConditionRelationshipUpdateParamList 场景关系更新参数
     * @throws com.clife.commons.base.exception.expert.ExpertServiceException
     */
    void setCondtitionRelationBatch(List<UserConditionInstance> userConditionRelationshipUpdateParamList);    
	```
	
	* 备注：

	如果添加条件包含地点圈，则提交的条件实例conditionInstance需包含circleId属性
	
3. 用户动作配置相关接口

	* 服务：UserActionsServiceFacade
	* 方法：
	
	```
	
	  /**
     * 获取用户场景动作列表
     * @param userActions 用户动作参数
     * @return 用户场景动作列表
     */
    List<RelUserSceneDevice> actionsList(UserActions userActions);

    /**
     * 新增/修改用户动作（设备类）
     * @param userActions
     */
    UserActions insertOrUpdate(UserActions userActions);


    /**
     * 批量删除用户动作
     * @param userActionsIdList 用户动作ID列表
     * @return
     * @throws
     */
    void deleteUserActionsBatch(List<Integer> userActionsIdList);



    /**
     * 批量新增动作（设备类且同一子场景），根据userActionsId是否为空决定是新增还是修改，
     * 新增的数据不会影响当前子场景已存在的数据
     * @param userActionses
     * @param deviceInfoMap 动作列表中相关设备的设备信息
     * @param userSceneId
     * @param subSceneIndex
     * @return code:103003039 同一延迟时间动作已存在;code:0 成功
     */
    public ResultStatus insertOrUpdateBatch(Integer userId, String appId, List<UserActions> userActionsList, Long userSceneId, Integer subSceneIndex);

    
	```

--

#### 地点事件上报

* 通信方式： MQ
* 消息协议：


| 参数  | 类型  | 含义 |
|:----:|:----:|:----|
| userId | int | 用户id |
| circle | string | 地点圈id |
| value | string | 进入或离开地点圈：1 进入， 0 离开 |

* 调用举例：

```
	Map mapMsg = new HashMap();
	mapMsg.setString("circle", "7");
	mapMsg.setInt("userId", 1857);
	mapMsg.setString("value", "1");

	ExpertMsgSender msgSender = new LBSExpertMsgSender();
	msgSender.sendMsg(mapMsg);

```









