---
layout: post
title: Connector分析-tomcat6.x源码阅读
date: 2013-11-16
categories: java
---
**2013-11-16**


在tomcate中需要解决响应客户端请求的Socket问题，即接收客户端的请求，Connector作为tomcate中的链接器，管理负责监控网络端口的网络连接器。它是以管理者的身份存在，不涉及具体的网络接口监听，只负责管理监听网络组件，负责组件的所需资源的调配和组件的运行状态控制。作为一个壳，它能管理实现ProtocolHandler接口的网络监听组件，能方便替换网络监听组件。

Connector作为一个tomcate组件的生命周期中应该经历这样一个过程:初始化-启动-停止-销毁.作为Connector，它需要解决一些问题

*监听网络端口组件*  
在带一个参数的构造器中指定监听网络端口组件，传递协议版本或者类名，setProtocol(String)方法负责辨别和之人协议监听网络端口的组件，在构造器中实例化监控网络端口组件。

```java

	/**
	 * 指定协议类型
	 * @param protocol
	 * @throws Exception
	 */
	public Connector(String protocol) throws Exception {
		setProtocol(protocol);
		// Instantiate protocol handler
		try {
			Class clazz = Class.forName(protocolHandlerClassName);
			this.protocolHandler = (ProtocolHandler) clazz.newInstance();
		} catch (Exception e) {
			log.error(sm.getString(
					"coyoteConnector.protocolHandlerInstantiationFailed", e));
		}
	}

```

Connector默认的网络端口监听组件是org.apache.coyote.http11.Http11Protocol，同时还附加指定一些常量配置。在完成指定组件后，Connector需要完成组件的初始化任务。

*init()*  
负责完成Connector组件的初始化任务，首先先判断Connector所依附的Service是不为空，然后判断所寄居的容器不为空，若为空，findContainer()方法通过JMX的方式来查询所寄居的容器，同时在方法内调用真正初始化方法initialize()。

*initialize()*  
真正负责完成组件的初始化任务。在方法内部主要完成下面几步:

+ 注册组件到JMX
+ 设置网络监听端口组件的适配器(CoyoteAdapter)
+ 反射为网络监听组件设置jkHome
+ 网络监听端口组件初始化

```java

	/**
	 * Initialize this connector (create ServerSocket here!)
	 * 初始化连接器，创建ServerSocket
	 */
	public void initialize() throws LifecycleException {
		if (initialized) {
			if (log.isInfoEnabled())
				log.info(sm.getString("coyoteConnector.alreadyInitialized"));
			return;
		}

		this.initialized = true;

		if (oname == null && (container instanceof StandardEngine)) {
			try {
				// we are loaded directly, via API - and no name was given to us
				StandardEngine cb = (StandardEngine) container;
				oname = createObjectName(cb.getName(), "Connector");
				Registry.getRegistry(null, null).registerComponent(this, oname,
						null);
				controller = oname;
			} catch (Exception e) {
				log.error("Error registering connector ", e);
			}
			if (log.isDebugEnabled())
				log.debug("Creating name for connector " + oname);
		}

		// Initializa adapter
		//初始化适配器
		adapter = new CoyoteAdapter(this);
		//设置协议处理的适配器
		protocolHandler.setAdapter(adapter);

		// Make sure parseBodyMethodsSet has a default
		if (null == parseBodyMethodsSet)
			setParseBodyMethods(getParseBodyMethods());

		IntrospectionUtils.setProperty(protocolHandler, "jkHome",
				System.getProperty("catalina.base"));

		try {
			protocolHandler.init();//协议处理器初始化
		} catch (Exception e) {
			throw new LifecycleException(sm.getString(
					"coyoteConnector.protocolHandlerInitializationFailed", e));
		}
	}
```
从上面可以看出，初始化是在做资源配置工作.在组件初始化化完成后，就是组件启动的任务了，start()方法负责这个功能。

*start()*
负责组件的启动任务，在方法内部做些事件通知和启动网络监听端口组件.

+ 判断组件是否已经初始化和启动
+ 触发START_EVENT生命周期事件监听，标记已经启动
+ 注册组件到JMX
+ 启动网络监听端口组件

```java

	/**
	 * Begin processing requests via this Connector.
	 * 开始监听请求
	 * 
	 * @exception LifecycleException
	 *                if a fatal startup error occurs
	 */
	public void start() throws LifecycleException {
		if (!initialized)
			initialize();

		// Validate and update our current state
		if (started) {
			if (log.isInfoEnabled())
				log.info(sm.getString("coyoteConnector.alreadyStarted"));
			return;
		}
		lifecycle.fireLifecycleEvent(START_EVENT, null);
		started = true;

		// We can't register earlier - the JMX registration of this happens
		// in Server.start callback
		if (this.oname != null) {
			// We are registred - register the adapter as well.
			try {
				Registry.getRegistry(null, null).registerComponent(
						protocolHandler,
						createObjectName(this.domain, "ProtocolHandler"), null);
			} catch (Exception ex) {
				log.error(
						sm.getString("coyoteConnector.protocolRegistrationFailed"),
						ex);
			}
		} else {
			if (log.isInfoEnabled())
				log.info(sm.getString("coyoteConnector.cannotRegisterProtocol"));
		}

		try {
			protocolHandler.start();//协议监听
		} catch (Exception e) {
			String errPrefix = "";
			if (this.service != null) {
				errPrefix += "service.getName(): \"" + this.service.getName()
						+ "\"; ";
			}

			throw new LifecycleException(errPrefix
					+ " "
					+ sm.getString(
							"coyoteConnector.protocolHandlerStartFailed", e));
		}

		if (this.domain != null) {
			mapperListener.setDomain(domain);
			// mapperListener.setEngine( service.getContainer().getName() );
			mapperListener.init();
			try {
				ObjectName mapperOname = createObjectName(this.domain, "Mapper");
				if (log.isDebugEnabled())
					log.debug(sm.getString(
							"coyoteConnector.MapperRegistration", mapperOname));
				Registry.getRegistry(null, null).registerComponent(mapper,
						mapperOname, "Mapper");
			} catch (Exception ex) {
				log.error(
						sm.getString("coyoteConnector.protocolRegistrationFailed"),
						ex);
			}
		}
	}

```

启动完成后组件监听网络端口，对网络监听端口组件有暂定监听的需求和回复监听需求，通过`pause()`和`resume()`完成，方法内部负责调用组件的暂停监听和恢复监听方法.组件的停止运行由`stop()`完成.

*stop()*  
负责组件的停止功能，也即停止网络监听端口组件。

+ 判断是否启动
+ 触发STOP_EVENT事件，标记没有启动状态
+ 撤销组件的JMX注册
+ 调用网络监听端口组件的destory()方法

```java

	public void stop() throws LifecycleException {

		// Validate and update our current state
		if (!started) {
			log.error(sm.getString("coyoteConnector.notStarted"));
			return;

		}
		lifecycle.fireLifecycleEvent(STOP_EVENT, null);
		started = false;

		try {
			mapperListener.destroy();
			Registry.getRegistry(null, null).unregisterComponent(
					createObjectName(this.domain, "Mapper"));
			Registry.getRegistry(null, null).unregisterComponent(
					createObjectName(this.domain, "ProtocolHandler"));
		} catch (MalformedObjectNameException e) {
			log.error(sm
					.getString("coyoteConnector.protocolUnregistrationFailed"),
					e);
		}
		try {
			protocolHandler.destroy();
		} catch (Exception e) {
			throw new LifecycleException(sm.getString(
					"coyoteConnector.protocolHandlerDestroyFailed", e));
		}

	}
```

以上几步即完成了组件的stop任务，组件的销毁工作交由destory()完成，所做的工作就是撤销JMX注册和Service移除Connector组件

Connector作为tomcate的连接器组件，负责网络端口监听组件的管理配置，能管理实现ProtocolHandler接口，在不需要修改代码的情况下，方便实现网络端口监听组件的替换，对组件管理和代码维护很有帮助.
