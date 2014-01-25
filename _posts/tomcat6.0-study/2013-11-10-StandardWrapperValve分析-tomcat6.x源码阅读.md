---
layout: post
title: StandardWrapperValve分析-tomcat6.x源码阅读
date: 2013-11-10
categories: java
---
**2013-11-10**

StandardWrapperValve是StandardWrapper容器的BasicValve，tomcat使用容器的BasicValve来控制处理请求，StandardWrapperValve的作用是负责为请求选择Wrapper,调用Servlet处理请求,控制请求处理流程。寄生于StandardWrapper中，StandardWrapperValve作为BasicValve是如何完成即可功能的呢。

StandardWrapperValve在方法invoke(Request, Response)中完成即可功能,在StandardCOntextValve的invoke(Request, Response)方法中调用。invoke(Request, Response)中做了很多逻辑操作，方法被标记为final，不允许更改方法。

**invoke(Request, Response)**  
负责选择Wrapper过滤Filter，调用servlet处理请求，在方法中有很多逻辑操作，详细如下:

+ 验证Wrapper和Context的可用性
+ 通过wrapper的allocate()方法来获取servlet
+ 判断是否为Comet
+ 调用response.sendAcknowledgement();发送"HTTP/1.1 100 Continue" + CRLF + CRLF
+ 获取在该Servlet上面注册的所有Filter，组成FilterChain
+ 如果为JSP，设置属性
+ 在Servlet上面调用FilterChain过滤，在FilterChain链完成对Servlet方法的调用，处理请求
+ 重置FilterChain链
+ 调用wrapper.deallocate(servlet);归还Wrapper
+ 判断Servlet还是否可以用，如果不可用，调用wrapper.unload();卸载Servlet

```java

	/**
	 * Process a Comet event. The main differences here are to not use sendError
	 * (the response is committed), to avoid creating a new filter chain (which
	 * would work but be pointless), and a few very minor tweaks.
	 * 
	 * @param request
	 *            The servlet request to be processed
	 * @param response
	 *            The servlet response to be created
	 * 
	 * @exception IOException
	 *                if an input/output error occurs, or is thrown by a
	 *                subsequently invoked Valve, Filter, or Servlet
	 * @exception ServletException
	 *                if a servlet error occurs, or is thrown by a subsequently
	 *                invoked Valve, Filter, or Servlet
	 */
	public void event(Request request, Response response, CometEvent event)
			throws IOException, ServletException {

		// Initialize local variables we may need
		Throwable throwable = null;
		// This should be a Request attribute...
		long t1 = System.currentTimeMillis();
		// FIXME: Add a flag to count the total amount of events processed ?
		// requestCount++;
		StandardWrapper wrapper = (StandardWrapper) getContainer();
		Servlet servlet = null;
		Context context = (Context) wrapper.getParent();

		// Check for the application being marked unavailable
		boolean unavailable = !context.getAvailable()
				|| wrapper.isUnavailable();

		// Allocate a servlet instance to process this request
		try {
			if (!unavailable) {
				servlet = wrapper.allocate();
			}
		} catch (UnavailableException e) {
			// The response is already committed, so it's not possible to do
			// anything
		} catch (ServletException e) {
			container
					.getLogger()
					.error(sm.getString("standardWrapper.allocateException",
							wrapper.getName()), StandardWrapper.getRootCause(e));
			throwable = e;
			exception(request, response, e);
			servlet = null;
		} catch (Throwable e) {
			container.getLogger().error(
					sm.getString("standardWrapper.allocateException",
							wrapper.getName()), e);
			throwable = e;
			exception(request, response, e);
			servlet = null;
		}

		MessageBytes requestPathMB = null;
		if (request != null) {
			requestPathMB = request.getRequestPathMB();
		}
		request.setAttribute(ApplicationFilterFactory.DISPATCHER_TYPE_ATTR,
				ApplicationFilterFactory.REQUEST_INTEGER);
		request.setAttribute(
				ApplicationFilterFactory.DISPATCHER_REQUEST_PATH_ATTR,
				requestPathMB);
		// Get the current (unchanged) filter chain for this request
		ApplicationFilterChain filterChain = (ApplicationFilterChain) request
				.getFilterChain();

		// Call the filter chain for this request
		// NOTE: This also calls the servlet's event() method
		try {
			String jspFile = wrapper.getJspFile();
			if (jspFile != null)
				request.setAttribute(Globals.JSP_FILE_ATTR, jspFile);
			else
				request.removeAttribute(Globals.JSP_FILE_ATTR);
			if ((servlet != null) && (filterChain != null)) {

				// Swallow output if needed
				if (context.getSwallowOutput()) {
					try {
						SystemLogHandler.startCapture();
						filterChain.doFilterEvent(request.getEvent());
					} finally {
						String log = SystemLogHandler.stopCapture();
						if (log != null && log.length() > 0) {
							context.getLogger().info(log);
						}
					}
				} else {
					filterChain.doFilterEvent(request.getEvent());
				}

			}
			request.removeAttribute(Globals.JSP_FILE_ATTR);
		} catch (ClientAbortException e) {
			request.removeAttribute(Globals.JSP_FILE_ATTR);
			throwable = e;
			exception(request, response, e);
		} catch (IOException e) {
			request.removeAttribute(Globals.JSP_FILE_ATTR);
			container.getLogger().error(
					sm.getString("standardWrapper.serviceException",
							wrapper.getName()), e);
			throwable = e;
			exception(request, response, e);
		} catch (UnavailableException e) {
			request.removeAttribute(Globals.JSP_FILE_ATTR);
			container.getLogger().error(
					sm.getString("standardWrapper.serviceException",
							wrapper.getName()), e);
			// Do not save exception in 'throwable', because we
			// do not want to do exception(request, response, e) processing
		} catch (ServletException e) {
			request.removeAttribute(Globals.JSP_FILE_ATTR);
			Throwable rootCause = StandardWrapper.getRootCause(e);
			if (!(rootCause instanceof ClientAbortException)) {
				container.getLogger().error(
						sm.getString("standardWrapper.serviceException",
								wrapper.getName()), rootCause);
			}
			throwable = e;
			exception(request, response, e);
		} catch (Throwable e) {
			request.removeAttribute(Globals.JSP_FILE_ATTR);
			container.getLogger().error(
					sm.getString("standardWrapper.serviceException",
							wrapper.getName()), e);
			throwable = e;
			exception(request, response, e);
		}

		// Release the filter chain (if any) for this request
		if (filterChain != null) {
			filterChain.reuse();
		}

		// Deallocate the allocated servlet instance
		try {
			if (servlet != null) {
				wrapper.deallocate(servlet);
			}
		} catch (Throwable e) {
			container.getLogger().error(
					sm.getString("standardWrapper.deallocateException",
							wrapper.getName()), e);
			if (throwable == null) {
				throwable = e;
				exception(request, response, e);
			}
		}

		// If this servlet has been marked permanently unavailable,
		// unload it and release this instance
		try {
			if ((servlet != null) && (wrapper.getAvailable() == Long.MAX_VALUE)) {
				wrapper.unload();
			}
		} catch (Throwable e) {
			container.getLogger().error(
					sm.getString("standardWrapper.unloadException",
							wrapper.getName()), e);
			if (throwable == null) {
				throwable = e;
				exception(request, response, e);
			}
		}

		long t2 = System.currentTimeMillis();

		long time = t2 - t1;
		processingTime += time;
		if (time > maxTime)
			maxTime = time;
		if (time < minTime)
			minTime = time;

	}

```

以上是invoke(Request,Response)方法的源码，代码中发现有很多的异常处理逻辑，针对不同的异常处理逻辑，响应给请求的内容页不一样。从上面可以看出invoke方法控制了请求的传递过程，在方法并没有直接调用servlet的方法，而是将调用servlet中的方法放到FilterChain中。
