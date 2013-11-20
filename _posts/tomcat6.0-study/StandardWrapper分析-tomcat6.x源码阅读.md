**2013-10-20**

StandardWrapper是什么
StandardWrapper是负责对Servlet的封装，在tomcat的结构层次中属于最内层，跟Servlet最接近的组件，是装载Servlet的容器，StandardWrapper没有子容器，因为不支持addChild()方法。在StandardWrapper之前Host已经解决了Servlet的选择问题，那么StandardWrapper要解决的问题是加载实例化Servlet，并使之可用，能够调用开发者定义的请求处理逻辑响应请求，且要维护Servlet的上面周期，资源分配。首先先看StandardWrapper里面所具备的信息。

**Wrapper**  
StandardWrapper的实现接口，是包装Servlet的接口，定义了加载Servlet、使用Servlet和管理维护Servlet的方法，包装类主要通过java反射方式动态加载Context所需要的Servlet，并将Servlet缓存一段时间，两个重要的方法：load()方法用来加载Servlet，allocate()方法用来完成Servlet使用前的准备工作。

**ContainerBase**  
被StandardWrapper继承容器基类，赋予StandardWrapper容器功能。

**StandardWrapper()**  
默认构造方法，在构造方法中，与其他容器一样都指定pipeline的basic类型:StandardWrapperValve,在pipeline链尾部，控制请求数据的流动，在StandardWrapper的功能是调用StandardWrapper获取Servlet通过反射调用Servlet方法响应请求完成任务。

**facade : StandardWrapperFacade**  
是设计模式中外观模式的应用，对ServletConfig封装，简化Servlet对ServletConifg的访问。

**servletClass : String**  
StandardWrapper包装的对象，servlet类文件全报包名，用于记录包装类。

**swValve : StandardWrapperValve**  
pipeline中的basicValve，作用很大，主要功能是使用servlet的逻辑处理请求信息。

**getServletMethods()**  
工具方法，在使用servlet中需要知道servlet所具有的方法

上面内容都是一些类配置信息，下面的内容是关于StandardWrapper如何加载servlet并管理servlet的。首先是从磁盘上读取servlet文件加载到虚拟机中，这步由**loadServlet()**完成

**loadServlet()**  
使用这个方法来将磁盘上的servlet类文件加载到JVM中，并完成实例化。在loadServlet中主要步骤如下：

+ 判断是否是标记为单线程模式(该模式有性能问题，每次请求都会生成实例)和是否已经初始化过
+ 判断是否为jsp文件，如果是jsp,按jsp处理方法处理
+ 判断是否有servlet类文件
+ 获取类加载器并判断是否获取成功，用于加载servlet
+ 使用类加载装载servlet字节码到JVM中
+ 实例化servlet
+ 校验servlet是否是Httpservlet子类 
+ 触发BEFORE_INIT_EVENT事件
+ 调用init()初始化servlet
+ JSP文件判断处理，模拟触发调用service方法
+ 触发AFTER_INIT_EVENT事件
+ 触发load事件

至此，StandardWrapper已经完成servlet的装载和实例化操作，下面是loadServlet()方法源码

```java

	/**
	 * Load and initialize an instance of this servlet, if there is not already
	 * at least one initialized instance. This can be used, for example, to load
	 * servlets that are marked in the deployment descriptor to be loaded at
	 * server startup time.
	 * 装载和初始化一个servlet实例，
	 */
	public synchronized Servlet loadServlet() throws ServletException {

		// Nothing to do if we already have an instance or an instance pool
		if (!singleThreadModel && (instance != null))
			return instance;

		PrintStream out = System.out;
		if (swallowOutput) {
			SystemLogHandler.startCapture();
		}

		Servlet servlet;
		try {
			long t1 = System.currentTimeMillis();
			// If this "servlet" is really a JSP file, get the right class.
			// HOLD YOUR NOSE - this is a kludge that avoids having to do
			// special
			// case Catalina-specific code in Jasper - it also requires that the
			// servlet path be replaced by the <jsp-file> element content in
			// order to be completely effective
			String actualClass = servletClass;
			if ((actualClass == null) && (jspFile != null)) {
				Wrapper jspWrapper = (Wrapper) ((Context) getParent())
						.findChild(Constants.JSP_SERVLET_NAME);
				if (jspWrapper != null) {
					actualClass = jspWrapper.getServletClass();
					// Merge init parameters
					String paramNames[] = jspWrapper.findInitParameters();
					for (int i = 0; i < paramNames.length; i++) {
						if (parameters.get(paramNames[i]) == null) {
							parameters
									.put(paramNames[i], jspWrapper
											.findInitParameter(paramNames[i]));
						}
					}
				}
			}

			// Complain if no servlet class has been specified
			if (actualClass == null) {
				unavailable(null);
				throw new ServletException(sm.getString(
						"standardWrapper.notClass", getName()));
			}

			// Acquire an instance of the class loader to be used
			Loader loader = getLoader();
			if (loader == null) {
				unavailable(null);
				throw new ServletException(sm.getString(
						"standardWrapper.missingLoader", getName()));
			}

			ClassLoader classLoader = loader.getClassLoader();

			// Special case class loader for a container provided servlet
			//
			if (isContainerProvidedServlet(actualClass)
					&& !((Context) getParent()).getPrivileged()) {
				// If it is a priviledged context - using its own
				// class loader will work, since it's a child of the container
				// loader
				classLoader = this.getClass().getClassLoader();
			}

			// Load the specified servlet class from the appropriate class
			// loader
			Class classClass = null;
			try {
				if (SecurityUtil.isPackageProtectionEnabled()) {
					final ClassLoader fclassLoader = classLoader;
					final String factualClass = actualClass;
					try {
						classClass = (Class) AccessController
								.doPrivileged(new PrivilegedExceptionAction() {
									public Object run() throws Exception {
										if (fclassLoader != null) {
											return fclassLoader
													.loadClass(factualClass);
										} else {
											return Class.forName(factualClass);
										}
									}
								});
					} catch (PrivilegedActionException pax) {
						Exception ex = pax.getException();
						if (ex instanceof ClassNotFoundException) {
							throw (ClassNotFoundException) ex;
						} else {
							getServletContext().log(
									"Error loading " + fclassLoader + " "
											+ factualClass, ex);
						}
					}
				} else {
					if (classLoader != null) {
						classClass = classLoader.loadClass(actualClass);
					} else {
						classClass = Class.forName(actualClass);
					}
				}
			} catch (ClassNotFoundException e) {
				unavailable(null);
				getServletContext().log(
						"Error loading " + classLoader + " " + actualClass, e);
				throw new ServletException(sm.getString(
						"standardWrapper.missingClass", actualClass), e);
			}

			if (classClass == null) {
				unavailable(null);
				throw new ServletException(sm.getString(
						"standardWrapper.missingClass", actualClass));
			}

			// Instantiate and initialize an instance of the servlet class
			// itself
			try {
				servlet = (Servlet) classClass.newInstance();
				// Annotation processing
				if (!((Context) getParent()).getIgnoreAnnotations()) {
					if (getParent() instanceof StandardContext) {
						((StandardContext) getParent())
								.getAnnotationProcessor().processAnnotations(
										servlet);
						((StandardContext) getParent())
								.getAnnotationProcessor()
								.postConstruct(servlet);
					}
				}
			} catch (ClassCastException e) {
				unavailable(null);
				// Restore the context ClassLoader
				throw new ServletException(sm.getString(
						"standardWrapper.notServlet", actualClass), e);
			} catch (Throwable e) {
				unavailable(null);

				// Added extra log statement for Bugzilla 36630:
				// http://issues.apache.org/bugzilla/show_bug.cgi?id=36630
				if (log.isDebugEnabled()) {
					log.debug(sm.getString("standardWrapper.instantiate",
							actualClass), e);
				}

				// Restore the context ClassLoader
				throw new ServletException(sm.getString(
						"standardWrapper.instantiate", actualClass), e);
			}

			// Check if loading the servlet in this web application should be
			// allowed
			if (!isServletAllowed(servlet)) {
				throw new SecurityException(sm.getString(
						"standardWrapper.privilegedServlet", actualClass));
			}

			// Special handling for ContainerServlet instances
			if ((servlet instanceof ContainerServlet)
					&& (isContainerProvidedServlet(actualClass) || ((Context) getParent())
							.getPrivileged())) {
				((ContainerServlet) servlet).setWrapper(this);
			}

			classLoadTime = (int) (System.currentTimeMillis() - t1);
			// Call the initialization method of this servlet
			try {
				instanceSupport.fireInstanceEvent(
						InstanceEvent.BEFORE_INIT_EVENT, servlet);

				if (Globals.IS_SECURITY_ENABLED) {
					boolean success = false;
					try {
						Object[] args = new Object[] { facade };
						SecurityUtil.doAsPrivilege("init", servlet, classType,
								args);
						success = true;
					} finally {
						if (!success) {
							// destroy() will not be called, thus clear the
							// reference now
							SecurityUtil.remove(servlet);
						}
					}
				} else {
					servlet.init(facade);
				}

				// Invoke jspInit on JSP pages
				if ((loadOnStartup >= 0) && (jspFile != null)) {
					// Invoking jspInit
					DummyRequest req = new DummyRequest();
					req.setServletPath(jspFile);
					req.setQueryString(Constants.PRECOMPILE + "=true");
					DummyResponse res = new DummyResponse();

					if (Globals.IS_SECURITY_ENABLED) {
						Object[] args = new Object[] { req, res };
						SecurityUtil.doAsPrivilege("service", servlet,
								classTypeUsedInService, args);
						args = null;
					} else {
						servlet.service(req, res);
					}
				}
				instanceSupport.fireInstanceEvent(
						InstanceEvent.AFTER_INIT_EVENT, servlet);
			} catch (UnavailableException f) {
				instanceSupport.fireInstanceEvent(
						InstanceEvent.AFTER_INIT_EVENT, servlet, f);
				unavailable(f);
				throw f;
			} catch (ServletException f) {
				instanceSupport.fireInstanceEvent(
						InstanceEvent.AFTER_INIT_EVENT, servlet, f);
				// If the servlet wanted to be unavailable it would have
				// said so, so do not call unavailable(null).
				throw f;
			} catch (Throwable f) {
				getServletContext().log("StandardWrapper.Throwable", f);
				instanceSupport.fireInstanceEvent(
						InstanceEvent.AFTER_INIT_EVENT, servlet, f);
				// If the servlet wanted to be unavailable it would have
				// said so, so do not call unavailable(null).
				throw new ServletException(sm.getString(
						"standardWrapper.initException", getName()), f);
			}

			// Register our newly initialized instance
			singleThreadModel = servlet instanceof SingleThreadModel;
			if (singleThreadModel) {
				if (instancePool == null)
					instancePool = new Stack();
			}
			fireContainerEvent("load", this);

			loadTime = System.currentTimeMillis() - t1;
		} finally {
			if (swallowOutput) {
				String log = SystemLogHandler.stopCapture();
				if (log != null && log.length() > 0) {
					if (getServletContext() != null) {
						getServletContext().log(log);
					} else {
						out.println(log);
					}
				}
			}
		}
		return servlet;

	}
```

StandardWrapper在通过loadServlet()方法完成Servlet加载并实例化后,Servlet的实例已经可以使用,但必须将其标记为可用,才会被Context使用到。通过**allocate()**方法完成这个步骤。

**allocate()**  
这个方法负责获取被StandardWrapper包装的Servlet对象实例,为Servlet能够被使用做最后的操作,主要步骤如下:

+ 判断当前是否在卸载servlet,否则抛出异常
+ 判断是否已经实例化,如果没有则在同步代码中,调用loadServlet()方法实例化,更新Servlet被启用的次数
+ 针对singleThreadModel此类的servlet单独处理,每次请求过程都会生成一个新的实例，实例会被加到servlet对象池中

```java

	/**
	 * Allocate an initialized instance of this Servlet that is ready to have
	 * its <code>service()</code> method called. If the servlet class does not
	 * implement <code>SingleThreadModel</code>, the (only) initialized instance
	 * may be returned immediately. If the servlet class implements
	 * <code>SingleThreadModel</code>, the Wrapper implementation must ensure
	 * that this instance is not allocated again until it is deallocated by a
	 * call to <code>deallocate()</code>.
	 * 
	 * @exception ServletException
	 *                if the servlet init() method threw an exception
	 * @exception ServletException
	 *                if a loading error occurs
	 */
	public Servlet allocate() throws ServletException {

		// If we are currently unloading this servlet, throw an exception
		if (unloading)
			throw new ServletException(sm.getString(
					"standardWrapper.unloading", getName()));

		boolean newInstance = false;

		// If not SingleThreadedModel, return the same instance every time
		if (!singleThreadModel) {

			// Load and initialize our instance if necessary
			if (instance == null) {
				synchronized (this) {
					if (instance == null) {
						try {
							if (log.isDebugEnabled())
								log.debug("Allocating non-STM instance");

							instance = loadServlet();
							// For non-STM, increment here to prevent a race
							// condition with unload. Bug 43683, test case #3
							if (!singleThreadModel) {
								newInstance = true;
								countAllocated.incrementAndGet();
							}
						} catch (ServletException e) {
							throw e;
						} catch (Throwable e) {
							throw new ServletException(
									sm.getString("standardWrapper.allocate"), e);
						}
					}
				}
			}

			if (!singleThreadModel) {
				if (log.isTraceEnabled())
					log.trace("  Returning non-STM instance");
				// For new instances, count will have been incremented at the
				// time of creation
				if (!newInstance) {
					countAllocated.incrementAndGet();
				}
				return (instance);
			}
		}

		synchronized (instancePool) {

			while (countAllocated.get() >= nInstances) {
				// Allocate a new instance if possible, or else wait
				if (nInstances < maxInstances) {
					try {
						instancePool.push(loadServlet());
						nInstances++;
					} catch (ServletException e) {
						throw e;
					} catch (Throwable e) {
						throw new ServletException(
								sm.getString("standardWrapper.allocate"), e);
					}
				} else {
					try {
						instancePool.wait();
					} catch (InterruptedException e) {
						;
					}
				}
			}
			if (log.isTraceEnabled())
				log.trace("  Returning allocated STM instance");
			countAllocated.incrementAndGet();
			return (Servlet) instancePool.pop();

		}

	}

```
这个方法真实目的是获取StandardWrapper包装后的Servlet类实例,有获取状态Servlet的方法，那么回收Servlet的方法就由deallocate(Servlet)来完成，它的任务也很简单，就是将已经分配的Servlet个数减一.

**deallocate(Servlet)**  
方法负责"回收"Servlet,只做了一件事:

+ 将分配Servlet记录数减一(对于singleThreadModel需要从Servlet对象池中移除然后计数减一)

```java


	/**
	 * Return this previously allocated servlet to the pool of available
	 * instances. If this servlet class does not implement SingleThreadModel, no
	 * action is actually required.
	 * 
	 * @param servlet
	 *            The servlet to be returned
	 * 
	 * @exception ServletException
	 *                if a deallocation error occurs
	 */
	public void deallocate(Servlet servlet) throws ServletException {

		// If not SingleThreadModel, no action is required
		if (!singleThreadModel) {
			countAllocated.decrementAndGet();
			return;
		}

		// Unlock and free this instance
		synchronized (instancePool) {
			countAllocated.decrementAndGet();
			instancePool.push(servlet);
			instancePool.notify();
		}

	}
```

在了解StandardWrapper如何将servlet类文件加载到JVM并实例化后，接下来就需要了解加载控制流程了，StandardWrapper利用load()方法来控制servlet的初始化工作，在load()方法中只做了一件事:调用loadServlet()方法.
**load()**  

Servlet类在实例化用完之后，在一定的timeout过了之后，就需要清理相当于已经废弃的Servlet所占用的宝贵内存空间,unload()方法负责完成这个功能，下面看一下unload()方法如何完成.

**unload()**  
负责卸载清除Servlet所占用的内存空间,释放所占用资源。流程无非就是事件通知，标记状态，卸载Servlet，详细如下：

+ 判断Servlet是singleThreadModel且instance实例不为空则继续
+ 标记unloading为真,防止在请求allocate()方法时分配一个不可靠的servlet
+ 循环等待所有与给Servlet有关的任务完成(判断分配计算是否为0)
+ 触发BEFORE_DESTROY_EVENT事件，调用servlet.destory()方法，通知servlet释放资源
+ 若为singleThreadModel类型，则需要清除占用的对象池
+ 标记unloading为假，触发unload事件

```java

	/**
	 * Unload all initialized instances of this servlet, after calling the
	 * <code>destroy()</code> method for each instance. This can be used, for
	 * example, prior to shutting down the entire servlet engine, or prior to
	 * reloading all of the classes from the Loader associated with our Loader's
	 * repository.
	 * 
	 * @exception ServletException
	 *                if an exception is thrown by the destroy() method
	 */
	public synchronized void unload() throws ServletException {

		// Nothing to do if we have never loaded the instance
		if (!singleThreadModel && (instance == null))
			return;
		unloading = true;

		// Loaf a while if the current instance is allocated
		// (possibly more than once if non-STM)
		if (countAllocated.get() > 0) {
			int nRetries = 0;
			long delay = unloadDelay / 20;
			while ((nRetries < 21) && (countAllocated.get() > 0)) {
				if ((nRetries % 10) == 0) {
					log.info(sm.getString("standardWrapper.waiting",
							countAllocated.toString()));
				}
				try {
					Thread.sleep(delay);
				} catch (InterruptedException e) {
					;
				}
				nRetries++;
			}
		}

		PrintStream out = System.out;
		if (swallowOutput) {
			SystemLogHandler.startCapture();
		}

		// Call the servlet destroy() method
		try {
			instanceSupport.fireInstanceEvent(
					InstanceEvent.BEFORE_DESTROY_EVENT, instance);

			if (Globals.IS_SECURITY_ENABLED) {
				try {
					SecurityUtil.doAsPrivilege("destroy", instance);
				} finally {
					SecurityUtil.remove(instance);
				}
			} else {
				instance.destroy();
			}

			instanceSupport.fireInstanceEvent(
					InstanceEvent.AFTER_DESTROY_EVENT, instance);

			// Annotation processing
			if (!((Context) getParent()).getIgnoreAnnotations()) {
				((StandardContext) getParent()).getAnnotationProcessor()
						.preDestroy(instance);
			}

		} catch (Throwable t) {
			instanceSupport.fireInstanceEvent(
					InstanceEvent.AFTER_DESTROY_EVENT, instance, t);
			instance = null;
			instancePool = null;
			nInstances = 0;
			fireContainerEvent("unload", this);
			unloading = false;
			throw new ServletException(sm.getString(
					"standardWrapper.destroyException", getName()), t);
		} finally {
			// Write captured output
			if (swallowOutput) {
				String log = SystemLogHandler.stopCapture();
				if (log != null && log.length() > 0) {
					if (getServletContext() != null) {
						getServletContext().log(log);
					} else {
						out.println(log);
					}
				}
			}
		}

		// Deregister the destroyed instance
		instance = null;

		if (singleThreadModel && (instancePool != null)) {
			try {
				while (!instancePool.isEmpty()) {
					Servlet s = (Servlet) instancePool.pop();
					if (Globals.IS_SECURITY_ENABLED) {
						try {
							SecurityUtil.doAsPrivilege("destroy", s);
						} finally {
							SecurityUtil.remove(s);
						}
					} else {
						s.destroy();
					}
					// Annotation processing
					if (!((Context) getParent()).getIgnoreAnnotations()) {
						((StandardContext) getParent())
								.getAnnotationProcessor().preDestroy(s);
					}
				}
			} catch (Throwable t) {
				instancePool = null;
				nInstances = 0;
				unloading = false;
				fireContainerEvent("unload", this);
				throw new ServletException(sm.getString(
						"standardWrapper.destroyException", getName()), t);
			}
			instancePool = null;
			nInstances = 0;
		}

		singleThreadModel = false;

		unloading = false;
		fireContainerEvent("unload", this);

	}

```

看完源码后发现这个方法对于是singleThreadModel标记的Servlet才有效，其他无效,坑爹.loadServlet()在加载servlet类时需要验证类对象类型是否为允许类型,isServletAllowed(Object)方法完成这个功能，做法很简单，判断父类类型.
**isServletAllowed(Object)**  
在了解StandardWrapper如何装载实例化后Servlet,作为容器组件,StandardWrapper调用start()方法来激活组件。

**start()**  
负责激活StandardWrapper组件，使之可用，在start()中，简单调用super.start()即完成。

```java

	/**
	 * Start this component, pre-loading the servlet if the load-on-startup
	 * value is set appropriately.
	 * 
	 * @exception LifecycleException
	 *                if a fatal error occurs during startup
	 */
	public void start() throws LifecycleException {

		// Send j2ee.state.starting notification
		if (this.getObjectName() != null) {
			Notification notification = new Notification("j2ee.state.starting",
					this.getObjectName(), sequenceNumber++);
			broadcaster.sendNotification(notification);
		}

		// Start up this component
		super.start();

		if (oname != null)
			registerJMX((StandardContext) getParent());

		// Load and initialize an instance of this servlet if requested
		// MOVED TO StandardContext START() METHOD

		setAvailable(0L);

		// Send j2ee.state.running notification
		if (this.getObjectName() != null) {
			Notification notification = new Notification("j2ee.state.running",
					this.getObjectName(), sequenceNumber++);
			broadcaster.sendNotification(notification);
		}

	}

```

StandardWrapper使用stop()方法来完成销毁自身的功能。stop()方法中比start()的功能多了一个，就是调用unload()方法清理singleThreadModel标记的servlet对象集合。

**stop()**  
销毁StandardWrapper，清理servlet对象实例，停止组件.

```java

	/**
	 * Stop this component, gracefully shutting down the servlet if it has been
	 * initialized.
	 * 
	 * @exception LifecycleException
	 *                if a fatal error occurs during shutdown
	 */
	public void stop() throws LifecycleException {

		setAvailable(Long.MAX_VALUE);

		// Send j2ee.state.stopping notification
		if (this.getObjectName() != null) {
			Notification notification = new Notification("j2ee.state.stopping",
					this.getObjectName(), sequenceNumber++);
			broadcaster.sendNotification(notification);
		}

		// Shut down our servlet instance (if it has been initialized)
		try {
			unload();
		} catch (ServletException e) {
			getServletContext().log(
					sm.getString("standardWrapper.unloadException", getName()),
					e);
		}

		// Shut down this component
		super.stop();

		// Send j2ee.state.stoppped notification
		if (this.getObjectName() != null) {
			Notification notification = new Notification("j2ee.state.stopped",
					this.getObjectName(), sequenceNumber++);
			broadcaster.sendNotification(notification);
		}

		if (oname != null) {
			Registry.getRegistry(null, null).unregisterComponent(oname);

			// Send j2ee.object.deleted notification
			Notification notification = new Notification("j2ee.object.deleted",
					this.getObjectName(), sequenceNumber++);
			broadcaster.sendNotification(notification);
		}

		if (isJspServlet && jspMonitorON != null) {
			Registry.getRegistry(null, null).unregisterComponent(jspMonitorON);
		}

	}

```

以上是StandardWrapper包装Servlet的详细过程，负责装载Servlet到JVM中，并实例化Servlet使之可用，如果然后维护管理Servlet对象实例，包括分配Servlet对象和回收Servlet对象。StandardWrapper作为tomcat容器组件中最小的容器，它能共享它父容器的资源容器.



