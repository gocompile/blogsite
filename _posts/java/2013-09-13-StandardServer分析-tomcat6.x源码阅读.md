---
layout: post
title:  "StandardServer分析-tomcat 6.x 源码阅读"
date:   2013-09-13 19:50:00
categories: java
---
**2013-09-13**


####StandardServer是什么####

StandardServer标准实现Server接口，从tomcat结构层次图中知道，Server处于最外层，其他组件在其内部,起统领做，像一家公司的CEO，负责管理整个公司，Server代表完整的Servlet容器,管理维护Server和全局resource，在各个组件中共享StandardServer资源。在tomcat启动过程中由Catalina通过Digester库加载解析server.xml，创建StandardServer对象，并初始化和启动Server,Server将自己注册到JMX上面，通过tomcat管理页面查看Server状态。StandardServer除了实现Server接口以外，还使用下列组件来完成功能。

**Lifecycle**

StandardServer实现Lifecycle接口，Lifecycle是tomcat中关于组件生命周期状态监控操作监听的接口，通过Lifecycle提供的9个状态和5个方法，使得监控组件的状态更新和在组件不同生命周期阶段操作成为可能。  
9的状态：指示组件状态

+ BEFORE_INIT
+ AFTER_INIT
+ BEFORE_START
+ START
+ AFTER_START
+ BEFORE_STOP
+ STOP
+ AFTER_STOP
+ BEFORE_DESTROY
+ AFTER_DESTROY
+ PERIODIC
+ CONFIG_START
+ CONFIG_STOP

5个方法：  更新组件状态和添加监听组件状态变更通知组件

+ addLifecycleListener(LifecycleListener)
+ findLifecycleListeners()
+ removeLifecycleListener(LifecycleListener)
+ start()
+ stop()

addLifecycleListener(LifecycleListener)是注册LifecycleEvent事件监听器。LifecycleListener是监听Lifecycle组件状态变更触发的LifecycleEvent事件，当LifecycleListener监听到LifecycleEvent事件事件时，会调用LifecycleListener方法lifecycleEvent(LifecycleEvent)响应事件。

**MBeanRegistration**

StandardServer实现MBeanRegistration接口。MBeanRegistration接口是JMX的MBean方法的内容，实现该接口的目的是将StandardServer组件注册到JMX中，通过JMX可以实现对StandardServer的控制。

**LifecycleSupport**

StandardServer的属性，它的作用就是负责管理Lifecycle接口实现类的LifecycleListener，只有一个到参的构造器，必须在创建对象时传入Lifecycle，在StandardServer中创建对象时同时将StandardServer自身传入，   
`private LifecycleSupport lifecycle = new LifecycleSupport(this);`  
LifecycleSupport 管理注册在StandardServer上的监听器，当监听到LifecycleEvent，LifecycleSupport 马上调用方法fireLifecycleEvent(String, Object)遍历监听器响应事件，属性state指明了当前Lifecycle的状态。

**javax.naming.Context**

StandardServer的属性，是JNDI中的内容，不了解，只知道它提供命名服务，也就是根据名字获取对象以及对象属性信息等，通过给定资源路径，然后就可以获取资源路径下面对象，在server.xml中关于资源的配置，从配置信息中可以看到配置的是tomcat管理页面登陆账号权限信息。
    
```  

	<!-- Global JNDI resources Documentation at /docs/jndi-resources-howto.html -->
	<GlobalNamingResources>
		<!-- Editable user database that can also be used by UserDatabaseRealm 
			to authenticate users -->
		<Resource name="UserDatabase" auth="Container"
			type="org.apache.catalina.UserDatabase" description="User database that can be updated and saved"
			factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
			pathname="conf/tomcat-users.xml" />
	</GlobalNamingResources>
```  
。java和第三方交互时经常使用，例如数据库驱动，mysql的JDBC和ODBC,java不用关心其实现部分。[JNDI理解例子，来自博客lujin55](http://lujin55.iteye.com/blog/1492536)

**NamingContextListener**

StandardServer的属性，是JNDI中的内容，不了解，比较复杂，只知道是负责监听javax.naming.Context资源事件。NamingContextListener实现了LifecycleListener、ContainerListener、PropertyChangeListener3个接口，具备监听Lifecycle组件，Container组件、PropertyChange的事件能力。

**NamingResources**

StandardServer的属性，是JNDI中的内容，不了解，比较复杂，知道它管理命名资源，将要加载的资源封装成对象，可以直接从NamingResources获取对象了。

**PropertyChangeSupport**

StandardServer的属性，参照LifecycleSupport的理解不难看出，PropertyChangeSupport管理对象属性变化监听器，跟LifecycleSupport的使命一样，监听PropertyChangeEvent事件，然后响应。

StandardServer通过JNDI加载server.xml中配置的资源，完成资源配置，注册监听器来监听事件，同时将自己注册到ServerFactory
上。StandardServer再次完成资源加载后调用init()方法初始化，实践调用的是initialize()，在tomcat启动中，调用初始化方法是由Catalina来完成。在initialize()方法中分4步完成

+ 第一步首先判断是否已经初始化
+ 第二部触发INIT_EVENT事件，标记已经初始化(initialized = true)
+ 第三步注册到MBeanServer服务器，添加监控，可以在tomcat管理页面管理Server
+ 第四步遍历services数组，挨个调用initialize()初始化

```

	/**
	 * Invoke a pre-startup initialization. This is used to allow connectors to
	 * bind to restricted ports under Unix operating environments.
	 */
	public void initialize() throws LifecycleException {
		if (initialized) {
			log.info(sm.getString("standardServer.initialize.initialized"));
			return;
		}
		lifecycle.fireLifecycleEvent(INIT_EVENT, null);
		initialized = true;

		if (oname == null) {
			try {
				oname = new ObjectName("Catalina:type=Server");
				Registry.getRegistry(null, null).registerComponent(this, oname,
						null);
			} catch (Exception e) {
				log.error("Error registering ", e);
			}
		}

		// Register global String cache
		try {
			ObjectName oname2 = new ObjectName(oname.getDomain()
					+ ":type=StringCache");
			Registry.getRegistry(null, null).registerComponent(
					new StringCache(), oname2, null);
		} catch (Exception e) {
			log.error("Error registering ", e);
		}

		// Initialize our defined Services
		for (int i = 0; i < services.length; i++) {
			services[i].initialize();
		}
	}
```

StandardServer初始化完成后，调用start()方法启动Server，在start()方法分4步完成

+ 第一步首先判断Server是否已经启动
+ 第二部触发BEFORE_START_EVENT，START_EVENT事件，标记已经启动(started = true)
+ 第三步遍历services数组，挨个调用start()初始化
+ 第四步触发AFTER_START_EVENT事件,通知监听器

```

	/**
	 * Prepare for the beginning of active use of the public methods of this
	 * component. This method should be called before any of the public methods
	 * of this component are utilized. It should also send a LifecycleEvent of
	 * type START_EVENT to any registered listeners.
	 * 
	 * @exception LifecycleException
	 *                if this component detects a fatal error that prevents this
	 *                component from being used
	 */
	public void start() throws LifecycleException {

		// Validate and update our current component state
		if (started) {
			log.debug(sm.getString("standardServer.start.started"));
			return;
		}

		// Notify our interested LifecycleListeners
		lifecycle.fireLifecycleEvent(BEFORE_START_EVENT, null);

		lifecycle.fireLifecycleEvent(START_EVENT, null);
		started = true;

		// Start our defined Services
		synchronized (services) {//同步
			for (int i = 0; i < services.length; i++) {
				if (services[i] instanceof Lifecycle)
					((Lifecycle) services[i]).start();
				System.out.println(i+":"+services[i].hashCode());//自己加的，为了验证catalina是否在Server的services列表中，最后事实证明catalina不在列表中
			}
		}

		// Notify our interested LifecycleListeners
		lifecycle.fireLifecycleEvent(AFTER_START_EVENT, null);

	}

```

StandardServer完成启动后，Catalina在start()方法中调用await()方法，await()中调用StandardServer的await()，StandardServer的await()方法主要完成的功能非常简单：启动ServerSocket监听网络端口请求关闭Server的请求,从代码中可以看出StandardServer使用一个死循环不断监听端口，当接收到"SHUTDOWN"命令时跳出循环，回到Catalina的start()方法中。

```

	/**
	 * Wait until a proper shutdown command is received, then return. This keeps
	 * the main thread alive - the thread pool listening for http connections is
	 * daemon threads.
	 */
	public void await() {
		// Negative values - don't wait on port - tomcat is embedded or we just
		// don't like ports
		if (port == -2) {
			// undocumented yet - for embedding apps that are around, alive.
			return;
		}
		if (port == -1) {
			try {
				awaitThread = Thread.currentThread();//当前线程
				while (!stopAwait) {
					try {
						Thread.sleep(10000);
					} catch (InterruptedException ex) {
						// continue and check the flag
					}
				}
			} finally {
				awaitThread = null;
			}
			return;
		}

		// Set up a server socket to wait on
		try {
			awaitSocket = new ServerSocket(port, 1,
					InetAddress.getByName("localhost"));
		} catch (IOException e) {
			log.error("StandardServer.await: create[" + port + "]: ", e);
			return;
		}

		try {
			awaitThread = Thread.currentThread();//当前线程

			// Loop waiting for a connection and a valid command
			while (!stopAwait) {
				ServerSocket serverSocket = awaitSocket;
				if (serverSocket == null) {
					break;
				}

				// Wait for the next connection
				Socket socket = null;
				StringBuilder command = new StringBuilder();
				try {
					InputStream stream = null;
					try {
						socket = serverSocket.accept();
						socket.setSoTimeout(10 * 1000); // Ten seconds
						stream = socket.getInputStream();
					} catch (AccessControlException ace) {
						log.warn("StandardServer.accept security exception: "
								+ ace.getMessage(), ace);
						continue;
					} catch (IOException e) {
						if (stopAwait) {
							// Wait was aborted with socket.close()
							break;
						}
						log.error("StandardServer.await: accept: ", e);
						break;
					}

					// Read a set of characters from the socket
					int expected = 1024; // Cut off to avoid DoS attack
					while (expected < shutdown.length()) {
						if (random == null)
							random = new Random();
						expected += (random.nextInt() % 1024);
					}
					while (expected > 0) {
						int ch = -1;
						try {
							ch = stream.read();
						} catch (IOException e) {
							log.warn("StandardServer.await: read: ", e);
							ch = -1;
						}
						if (ch < 32) // Control character or EOF terminates loop
							break;
						command.append((char) ch);
						expected--;
					}
				} finally {
					// Close the socket now that we are done with it
					try {
						if (socket != null) {
							socket.close();
						}
					} catch (IOException e) {
						// Ignore
					}
				}

				// Match against our command string
				boolean match = command.toString().equals(shutdown);
				if (match) {
					break;
				} else
					log.warn("StandardServer.await: Invalid command '"
							+ command.toString() + "' received");
			}
		} finally {
			ServerSocket serverSocket = awaitSocket;
			awaitThread = null;
			awaitSocket = null;

			// Close the server socket and return
			if (serverSocket != null) {
				try {
					serverSocket.close();
				} catch (IOException e) {
					// Ignore
				}
			}
		}
	}
```


在StandardServer调出awaite()方法循环回到Catalina的start()方法中，紧跟在awaite()方法后的是stop()方法，也即停止Server的方法，在StandardServer的stop()方法中主要完成停止Server的任务，分5步完成

+ 第一步首先判断Server是否已经启动
+ 第二部触发BEFORE_STOP_EVENT，STOP_EVENT事件，标记已经停止(started = false)
+ 第三步遍历services数组，挨个调用stop()初始化
+ 第四步触发AFTER_STOP_EVENT事件,通知监听器
+ 第五步调用stopAwait(),关闭ServerSocket，调用interrupt()退出线程。

```

	/**
	 * Gracefully terminate the active use of the public methods of this
	 * component. This method should be the last one called on a given instance
	 * of this component. It should also send a LifecycleEvent of type
	 * STOP_EVENT to any registered listeners.
	 * 
	 * @exception LifecycleException
	 *                if this component detects a fatal error that needs to be
	 *                reported
	 */
	public void stop() throws LifecycleException {

		// Validate and update our current component state
		if (!started)
			return;

		// Notify our interested LifecycleListeners
		lifecycle.fireLifecycleEvent(BEFORE_STOP_EVENT, null);

		lifecycle.fireLifecycleEvent(STOP_EVENT, null);
		started = false;

		// Stop our defined Services
		for (int i = 0; i < services.length; i++) {
			if (services[i] instanceof Lifecycle)
				((Lifecycle) services[i]).stop();
		}

		// Notify our interested LifecycleListeners
		lifecycle.fireLifecycleEvent(AFTER_STOP_EVENT, null);

		stopAwait();

	}
```
至此，StandardServer的初始化，启动，停止的内容完成了，StandardServer的认识也清晰了很多，StandardServer先从server.xml中加载资源完成配置，提供监听StandardServer状态方法，已经才不同生命周期操作StandardServer；在初始化中触发事件，初始化services，注册到MBeanServer，标记已经初始化，在启动中，触发启动事件，启动services，启动完成后，在由Catalina触发awaite()方法，启动ServerSocket监听请求关闭Server命令，在停止Server stop()方法中，触发停止Server事件，停止services，清除资源。注册在StandardServer上的监听器有点复杂，以后在看看是什么回事。

经过检测Catalian不在StandardServer的services列表中，那Catalian为何要实现Service接口呢？  
感觉JNDI有点意思，有时间好好看看!


啊啊啊...终于写完了，好累哦，欢迎吐槽...

**专一而精，要记得坚持**

