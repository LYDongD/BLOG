## 事件发布机制

#### 组件

* EventObject, 事件，包含source属性，指向事件源，即事件发布者，例如ApplicationContext
* ApplicationEventPublisher，事件发布者，实际上会委托ApplicationEventMulticaster管理监听器以及发布事件
* ApplicationEventMulticaster， 事件监听器管理者以及发布者
    * taskExecutor，线程池，实现异步发布事件
    * errorHandler，错误处理器，可进行错误处理
    * 定义并实现监听器添加，删除，以及发布事件等方法
    * 通过ConcurrentHashMap 持有特定类型事件与监听器的映射，作为缓存；如果没有，遍历所有监听器(LinkedHashSet)，选择可以处理当前事件的监听器
* ApplicationListener<E extends ApplicationEvent> 事件监听器， 实现事件处理回调方法
    * GenericApplicationListener
        * supportsEventType, 监听器需要知道自己是否支持特定的事件类型
        * supportsSourceType， 监听器需要知道自己是否支持特定的事件源

#### 源码追踪

> ApplicationEventMulticaster发布事件逻辑


```
    @Override
	public void multicastEvent(final ApplicationEvent event, ResolvableType eventType) {
        //获取事件类型
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));

        //获取所有该事件类型对应的事件监听器，发布事件
		for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			Executor executor = getTaskExecutor();
            //如果线程池不为空，则异步发布事件
			if (executor != null) {
				executor.execute(new Runnable() {
					@Override
					public void run() {
						invokeListener(listener, event);
					}
				});
			}
			else {//同步发布事件
				invokeListener(listener, event);
			}
		}
	}


    //发布事件即调用监听器的回调方法，并考虑异常处理
    protected void invokeListener(ApplicationListener listener, ApplicationEvent event) {
		ErrorHandler errorHandler = getErrorHandler();
		if (errorHandler != null) {
			try {
				listener.onApplicationEvent(event);
			}
			catch (Throwable err) {
				errorHandler.handleError(err);
			}
		}
		else {
			listener.onApplicationEvent(event);
		}
	}

```

> 监听器接口定义

```

//定义监听器的回调方法
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {

	/**
	 * Handle an application event.
	 * @param event the event to respond to
	 */
	void onApplicationEvent(E event);

}

//定义监听器可监听的事件类型和事件源类型
public interface GenericApplicationListener extends ApplicationListener<ApplicationEvent>, Ordered {

	/**
	 * Determine whether this listener actually supports the given event type.
	 * @param eventType the event type (never {@code null})
	 */
	boolean supportsEventType(ResolvableType eventType);

	/**
	 * Determine whether this listener actually supports the given source type.
	 * @param sourceType the source type, or {@code null} if no source
	 */
	boolean supportsSourceType(Class<?> sourceType);

}

```

> 监听器的管理和缓存

AbstractApplicationEventMulticast:

* ListenerRetriever 容器封装，存储所有的监听器

```
    public final Set<ApplicationListener<?>> applicationListeners;

    public final Set<String> applicationListenerBeans;

    //通过hash set 存储监听器
    public ListenerRetriever(boolean preFiltered) {
	    this.applicationListeners = new LinkedHashSet<ApplicationListener<?>>();
	    this.applicationListenerBeans = new LinkedHashSet<String>();
	    this.preFiltered = preFiltered;
    }

    //获取所有监听器，返回一个LinkedList存储的监听器列表
    public Collection<ApplicationListener<?>> getApplicationListeners() {
			LinkedList<ApplicationListener<?>> allListeners = new LinkedList<ApplicationListener<?>>();
            //将hash set中的监听器添加到Linked list 中
			for (ApplicationListener<?> listener : this.applicationListeners) {
				allListeners.add(listener);
			}
            
            //将hash set中的监听器名取出，通过spring容器获取对应的监听器对象，添加到LinkedList中
			if (!this.applicationListenerBeans.isEmpty()) {
				BeanFactory beanFactory = getBeanFactory();
				for (String listenerBeanName : this.applicationListenerBeans) {
					try {
                        //从容器中根据bean name 获取对应监听器，不重复添加
						ApplicationListener<?> listener = beanFactory.getBean(listenerBeanName, ApplicationListener.class);
						if (this.preFiltered || !allListeners.contains(listener)) {
							allListeners.add(listener);
						}
					}
					catch (NoSuchBeanDefinitionException ex) {
						// Singleton listener instance (without backing bean definition) disappeared -
						// probably in the middle of the destruction phase
					}
				}
			}
        
            //根据order注解，调整监听器的顺序
			AnnotationAwareOrderComparator.sort(allListeners);
			return allListeners;
		}

```

* retrieverCache 缓存特定的ListenerRetriever

默认的ListenerRetriever是defaultRetriver，每次获取listener可根据特定的事件类型进行分组，缓存至不同的ListenerRetriever中

```

    //初始大小为64的concurrentHashMap
    final Map<ListenerCacheKey, ListenerRetriever> retrieverCache =
			new ConcurrentHashMap<ListenerCacheKey, ListenerRetriever>(64);


    //分组的逻辑
    private Collection<ApplicationListener<?>> retrieveApplicationListeners(
			ResolvableType eventType, Class<?> sourceType, ListenerRetriever retriever) {

		LinkedList<ApplicationListener<?>> allListeners = new LinkedList<ApplicationListener<?>>();
		Set<ApplicationListener<?>> listeners;
		Set<String> listenerBeans;

        //从defaultRetriver获取所有的监听器和监听器bean名
		synchronized (this.retrievalMutex) {
			listeners = new LinkedHashSet<ApplicationListener<?>>(this.defaultRetriever.applicationListeners);
			listenerBeans = new LinkedHashSet<String>(this.defaultRetriever.applicationListenerBeans);
		}

        //根据事件类型添加到当前分组
		for (ApplicationListener<?> listener : listeners) {
			if (supportsEvent(listener, eventType, sourceType)) {
				if (retriever != null) {
					retriever.applicationListeners.add(listener);
				}
				allListeners.add(listener);
			}
		}

        //根据事件类型，从spring容器中获取监听器，添加到当前分组
		if (!listenerBeans.isEmpty()) {
			BeanFactory beanFactory = getBeanFactory();
			for (String listenerBeanName : listenerBeans) {
				try {
					Class<?> listenerType = beanFactory.getType(listenerBeanName);
					if (listenerType == null || supportsEvent(listenerType, eventType)) {
						ApplicationListener<?> listener =
								beanFactory.getBean(listenerBeanName, ApplicationListener.class);
						if (!allListeners.contains(listener) && supportsEvent(listener, eventType, sourceType)) {
							if (retriever != null) {
								retriever.applicationListenerBeans.add(listenerBeanName);
							}
							allListeners.add(listener);
						}
					}
				}
				catch (NoSuchBeanDefinitionException ex) {
					// Singleton listener instance (without backing bean definition) disappeared -
					// probably in the middle of the destruction phase
				}
			}
		}

        //调整顺序并返回
		AnnotationAwareOrderComparator.sort(allListeners);
		return allListeners;
	}


    //缓存逻辑
    protected Collection<ApplicationListener<?>> getApplicationListeners(
			ApplicationEvent event, ResolvableType eventType) {

		Object source = event.getSource();
		Class<?> sourceType = (source != null ? source.getClass() : null);
		ListenerCacheKey cacheKey = new ListenerCacheKey(eventType, sourceType);

		// Quick check for existing entry on ConcurrentHashMap..., 优先从缓存获取分组的监听器持有者
		ListenerRetriever retriever = this.retrieverCache.get(cacheKey);
		if (retriever != null) {
			return retriever.getApplicationListeners();
		}

        //确保缓存安全的情况下更新缓存
		if (this.beanClassLoader == null ||
				(ClassUtils.isCacheSafe(event.getClass(), this.beanClassLoader) &&
						(sourceType == null || ClassUtils.isCacheSafe(sourceType, this.beanClassLoader)))) {
			// Fully synchronized building and caching of a ListenerRetriever
			synchronized (this.retrievalMutex) {
                //双重检查
				retriever = this.retrieverCache.get(cacheKey);
				if (retriever != null) {
					return retriever.getApplicationListeners();
				}

                //从default中获取并添加到当前分组
				retriever = new ListenerRetriever(true);
				Collection<ApplicationListener<?>> listeners =
						retrieveApplicationListeners(eventType, sourceType, retriever);
                
                //更新缓存
				this.retrieverCache.put(cacheKey, retriever);
				return listeners;
			}
		}
		else {
			// No ListenerRetriever caching -> no synchronization necessary
			return retrieveApplicationListeners(eventType, sourceType, null);
		}
	}


```
