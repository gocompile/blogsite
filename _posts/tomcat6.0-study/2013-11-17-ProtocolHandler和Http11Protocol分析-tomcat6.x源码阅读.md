---
layout: post
title: ProtocolHandler和Http11Protocol分析-tomcat6.x源码阅读
date: 2013-11-17
categories: java
---
**2013-11-17**

tomcat处理HTTP请求，需要有一个网络端口监听ServerSocket来完成任务。接口ProtocolHandler被设计成控制网络端口监听组件运行，负责组件的生命周期控制，这个接口不是规范网络端口监听组件的功能，而是负责维护组件的生命周期。从名字上面看，属于网络协议处理者，但它不负责这功能，而是将其交给org.apache.coyote.Adapter来完成，这么设计估计是为了方便维护和拓展新功能。Http11Protocol是ProtocolHandler接口的一个实现(是Connector中默认处理协议),被设计成用来处理HTTP1.1网络协议的请求,通过该类可以完成在某个网络端口上面的监听,同时以HTTP1.1的协议来解析请求内容，然后将请求传递到Connector所寄居的Container容器pipeline流水工作线上处理。

*Http11Protocol*  
实现ProtocolHandler接口,为HTTP1.1而设计的协议处理组件，负责管理Http11ConnectionHandler和JIoEndpoint,前者负责处理HTTP1.1的连接，包括维护请求连接队列，为连接分配线程，设置连接属性等信息；后者是真正负责监控网络端口的组件，启动一个ServerSocket,不间断监听来自客户端的请求。Http11Protocol还包含了与HTTP1.1相关的许多属性信息,3个很重要的属性是包含前面提供的两个，还有一个ServerSocketFactory，用于定制ServerSocket。

了解一下Http11Protocol的初始化,启动,停止,注销功能.看看在这些重要的步骤中如何处理Http11ConnectionHandler和JIoEndpoint

*init()*  
负责组件的初始化,在方法内部完成了对JIoEndpoint的资源配置.

+ 为JIoEndpoint设置连接管理器(Http11ConnectionHandler)
+ 判断ServerSocketFactory不为null,为JIoEndpoint设置ServerSocketFactory
+ 判断ServerSocketFactory不为null,为ServerSocketFactory设置属性值
+ JIoEndpoint初始化

```java

	public void init() throws Exception {
		endpoint.setName(getName());
		endpoint.setHandler(cHandler);//设置连接管理器

		// Verify the validity of the configured socket factory
		try {
			if (isSSLEnabled()) {
				sslImplementation = SSLImplementation
						.getInstance(sslImplementationName);
				socketFactory = sslImplementation.getServerSocketFactory();
				endpoint.setServerSocketFactory(socketFactory);
			} else if (socketFactoryName != null) {
				socketFactory = (ServerSocketFactory) Class.forName(
						socketFactoryName).newInstance();
				endpoint.setServerSocketFactory(socketFactory);
			}
		} catch (Exception ex) {
			log.error(sm.getString("http11protocol.socketfactory.initerror"),
					ex);
			throw ex;
		}
		//设置socketFactory属性
		if (socketFactory != null) {
			Iterator<String> attE = attributes.keySet().iterator();
			while (attE.hasNext()) {
				String key = attE.next();
				Object v = attributes.get(key);
				socketFactory.setAttribute(key, v);
			}
		}

		try {
			endpoint.init();//初始化
		} catch (Exception ex) {
			log.error(sm.getString("http11protocol.endpoint.initerror"), ex);
			throw ex;
		}
		if (log.isInfoEnabled())
			log.info(sm.getString("http11protocol.init", getName()));
	}
```
在`init()`方法中,为JIoEndpoint设置连接处理器,用于处理来自客户端的连接,判断并处理ServerSocketFactory,最后一步初始化JIoEndpoint.

*start()*  
负责组件的启动任务.完成JMX的注册和启动JIoEndpoint组件.

```java

	public void start() throws Exception {
		if (this.domain != null) {
			try {
				tpOname = new ObjectName(domain + ":" + "type=ThreadPool,name="
						+ getName());
				Registry.getRegistry(null, null).registerComponent(endpoint,
						tpOname, null);
			} catch (Exception e) {
				log.error("Can't register endpoint");
			}
			rgOname = new ObjectName(domain
					+ ":type=GlobalRequestProcessor,name=" + getName());
			Registry.getRegistry(null, null).registerComponent(cHandler.global,
					rgOname, null);
		}

		try {
			endpoint.start();
		} catch (Exception ex) {
			log.error(sm.getString("http11protocol.endpoint.starterror"), ex);
			throw ex;
		}
		if (log.isInfoEnabled())
			log.info(sm.getString("http11protocol.start", getName()));
	}

```

*pause()*和*resume()*和*destroy()*  
这3个方法都比较简单,分别控制JIoEndpoint暂定监听网络和恢复监听以及注销组件.

Http11Protocol组件的定位是管理JIoEndpoint,JIoEndpoint本身与具体协议无关,只负责监听网络端口并创建连接,如何解析请求内容则交由连接处理器来完成,例如Http11ConnectionHandler连接处理器，专门处理HTTP1.1协议连接。Http11Protocol管理JIoEndpoint,为JIoEndpoint提供连接处理器.

