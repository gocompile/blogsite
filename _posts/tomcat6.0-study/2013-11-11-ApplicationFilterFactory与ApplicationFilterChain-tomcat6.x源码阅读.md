---
layout: post
title: ApplicationFilterFactory与ApplicationFilterChain-tomcat6.x源码阅读
date: 2013-11-11
categories: java
---
**2013-11-11**

ApplicationFilterFactory和ApplicationFilterChain都是跟Filter相关的类，前者根据注册在Wrapper的Filter,经过筛选成产后者,后者是多个Filter集合的管理类。

**ApplicationFilterFactory**  
是个单例工厂类，负责为Servlet生成FilterChain,在createFilterChain(ServletRequest, Wrapper, Servlet)方法内部,通过URL与Filter URL过滤规则配置为Servlet生成FilterChain,具体如下：

+ 取dispatcher type，用于验证请求
+ 验证参数正确性，如验证request
+ 创建ApplicationFilterChain对象实例,为装载Filter做准备
+ 取Wrapper所在Context上的所以Filter定义
+ 按路径(url path)匹配规则筛选Filter，添加Filter到ApplicationChain中
+ 按Servlet Name匹配规则筛选Filter，添加Filter到ApplicationChian中

```java

	/**
	 * Construct and return a FilterChain implementation that will wrap the
	 * execution of the specified servlet instance. If we should not execute a
	 * filter chain at all, return <code>null</code>.
	 * 
	 * @param request
	 *            The servlet request we are processing
	 * @param servlet
	 *            The servlet instance to be wrapped
	 */
	public ApplicationFilterChain createFilterChain(ServletRequest request,
			Wrapper wrapper, Servlet servlet) {

		// get the dispatcher type
		int dispatcher = -1;
		if (request.getAttribute(DISPATCHER_TYPE_ATTR) != null) {
			Integer dispatcherInt = (Integer) request
					.getAttribute(DISPATCHER_TYPE_ATTR);
			dispatcher = dispatcherInt.intValue();
		}
		String requestPath = null;
		Object attribute = request.getAttribute(DISPATCHER_REQUEST_PATH_ATTR);

		if (attribute != null) {
			requestPath = attribute.toString();
		}

		HttpServletRequest hreq = null;
		if (request instanceof HttpServletRequest)
			hreq = (HttpServletRequest) request;
		// If there is no servlet to execute, return null
		if (servlet == null)
			return (null);

		boolean comet = false;

		// Create and initialize a filter chain object
		ApplicationFilterChain filterChain = null;
		if (request instanceof Request) {
			Request req = (Request) request;
			comet = req.isComet();
			if (Globals.IS_SECURITY_ENABLED) {
				// Security: Do not recycle
				filterChain = new ApplicationFilterChain();
				if (comet) {
					req.setFilterChain(filterChain);
				}
			} else {
				filterChain = (ApplicationFilterChain) req.getFilterChain();
				if (filterChain == null) {
					filterChain = new ApplicationFilterChain();
					req.setFilterChain(filterChain);
				}
			}
		} else {
			// Request dispatcher in use
			filterChain = new ApplicationFilterChain();
		}

		filterChain.setServlet(servlet);

		filterChain
				.setSupport(((StandardWrapper) wrapper).getInstanceSupport());

		// Acquire the filter mappings for this Context
		StandardContext context = (StandardContext) wrapper.getParent();
		FilterMap filterMaps[] = context.findFilterMaps();

		// If there are no filter mappings, we are done
		if ((filterMaps == null) || (filterMaps.length == 0))
			return (filterChain);

		// Acquire the information we will need to match filter mappings
		String servletName = wrapper.getName();

		// Add the relevant path-mapped filters to this filter chain
		for (int i = 0; i < filterMaps.length; i++) {
			if (!matchDispatcher(filterMaps[i], dispatcher)) {
				continue;
			}
			if (!matchFiltersURL(filterMaps[i], requestPath))
				continue;
			ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) context
					.findFilterConfig(filterMaps[i].getFilterName());
			if (filterConfig == null) {
				; // FIXME - log configuration problem
				continue;
			}
			boolean isCometFilter = false;
			if (comet) {
				try {
					isCometFilter = filterConfig.getFilter() instanceof CometFilter;
				} catch (Exception e) {
					// Note: The try catch is there because getFilter has a lot
					// of
					// declared exceptions. However, the filter is allocated
					// much
					// earlier
				}
				if (isCometFilter) {
					filterChain.addFilter(filterConfig);
				}
			} else {
				filterChain.addFilter(filterConfig);
			}
		}

		// Add filters that match on servlet name second
		for (int i = 0; i < filterMaps.length; i++) {
			if (!matchDispatcher(filterMaps[i], dispatcher)) {
				continue;
			}
			if (!matchFiltersServlet(filterMaps[i], servletName))
				continue;
			ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) context
					.findFilterConfig(filterMaps[i].getFilterName());
			if (filterConfig == null) {
				; // FIXME - log configuration problem
				continue;
			}
			boolean isCometFilter = false;
			if (comet) {
				try {
					isCometFilter = filterConfig.getFilter() instanceof CometFilter;
				} catch (Exception e) {
					// Note: The try catch is there because getFilter has a lot
					// of
					// declared exceptions. However, the filter is allocated
					// much
					// earlier
				}
				if (isCometFilter) {
					filterChain.addFilter(filterConfig);
				}
			} else {
				filterChain.addFilter(filterConfig);
			}
		}

		// Return the completed filter chain
		return (filterChain);

	}

```

以上ApplicationFilterFactory完成了生产ApplicationFilterChain的任务，为Wrapper(Servlet)从Context上面筛选Filter并添加到FilterChian中。在ApplicationFilterChain中，主要是doFilter(ServletRequest, ServletResponse)方法完成Filter过滤Serlvet且最后调用servlet的service()方法的功能。

**ApplicationFilterChain**
在ApplicationFilterFactory完成生产FilterChain的任务后，ApplicationFIlterChain的功能则是使用FilterChain来过滤Servlet，由于它实现了Filterchain接口，所以具有Filterchain的功能，并且最终调用servlet.service()方法，在doFilter(ServletRequest, ServletResponse)方法中调用internalDoFilter(ServletRequest, ServletResponse)来完成，首先使用Filter过滤Servlet，最后完成service方法的调用。具体如下:

+ 判断是否已经过滤到FilterChain链最后一个Filter，没有则继续调用doFilter(ServletRequest, ServletResponse)
+ 在最后一个Filter过滤完成之后，也即在递归方法的最内部中，调用servlet.service(request, response);执行servlet
+ 在完成service方法调用后，方法递归出来，继续执行每一层Filter中的逻辑定义

```java

	/**
	 * 调用doFilter方法递归过滤servlet，在递归最内部调用servlet.service()
	 * @param request
	 * @param response
	 * @throws IOException
	 * @throws ServletException
	 */
	private void internalDoFilter(ServletRequest request,
			ServletResponse response) throws IOException, ServletException {

		// Call the next filter if there is one
		//判断是否已经执行到FilterChain链尾部
		if (pos < n) {
			//取当前过滤Filter配置信息
			ApplicationFilterConfig filterConfig = filters[pos++];
			Filter filter = null;
			try {
				//取Filter
				filter = filterConfig.getFilter();
				//事件通知
				support.fireInstanceEvent(InstanceEvent.BEFORE_FILTER_EVENT,
						filter, request, response);
				//安全判断
				if (Globals.IS_SECURITY_ENABLED) {
					final ServletRequest req = request;
					final ServletResponse res = response;
					Principal principal = ((HttpServletRequest) req)
							.getUserPrincipal();

					Object[] args = new Object[] { req, res, this };
					//执行调用Filter的doFilter()方法
					SecurityUtil.doAsPrivilege("doFilter", filter, classType,
							args, principal);

					args = null;
				} else {
					//执行调用Filter的doFilter()方法
					filter.doFilter(request, response, this);
				}
				//完成Filter过滤事件通知
				support.fireInstanceEvent(InstanceEvent.AFTER_FILTER_EVENT,
						filter, request, response);
			} catch (IOException e) {
				if (filter != null)
					support.fireInstanceEvent(InstanceEvent.AFTER_FILTER_EVENT,
							filter, request, response, e);
				throw e;
			} catch (ServletException e) {
				if (filter != null)
					support.fireInstanceEvent(InstanceEvent.AFTER_FILTER_EVENT,
							filter, request, response, e);
				throw e;
			} catch (RuntimeException e) {
				if (filter != null)
					support.fireInstanceEvent(InstanceEvent.AFTER_FILTER_EVENT,
							filter, request, response, e);
				throw e;
			} catch (Throwable e) {
				if (filter != null)
					support.fireInstanceEvent(InstanceEvent.AFTER_FILTER_EVENT,
							filter, request, response, e);
				throw new ServletException(sm.getString("filterChain.filter"),
						e);
			}
			return;
		}

		// We fell off the end of the chain -- call the servlet instance
		try {
			//安全校验
			if (Globals.STRICT_SERVLET_COMPLIANCE) {
				lastServicedRequest.set(request);
				lastServicedResponse.set(response);
			}

			//调用servlet.service()事件通知
			support.fireInstanceEvent(InstanceEvent.BEFORE_SERVICE_EVENT,
					servlet, request, response);
			if ((request instanceof HttpServletRequest)
					&& (response instanceof HttpServletResponse)) {
				//安全校验
				if (Globals.IS_SECURITY_ENABLED) {
					final ServletRequest req = request;
					final ServletResponse res = response;
					Principal principal = ((HttpServletRequest) req)
							.getUserPrincipal();
					Object[] args = new Object[] { req, res };
					SecurityUtil.doAsPrivilege("service", servlet,
							classTypeUsedInService, args, principal);
					args = null;
				} else {
					//调用servlet.service()方法
					servlet.service((HttpServletRequest) request,
							(HttpServletResponse) response);
				}
			} else {
				//调用servlet.service()方法
				servlet.service(request, response);
			}
			//完成servlet.service()调用事件通知
			support.fireInstanceEvent(InstanceEvent.AFTER_SERVICE_EVENT,
					servlet, request, response);
		} catch (IOException e) {
			support.fireInstanceEvent(InstanceEvent.AFTER_SERVICE_EVENT,
					servlet, request, response, e);
			throw e;
		} catch (ServletException e) {
			support.fireInstanceEvent(InstanceEvent.AFTER_SERVICE_EVENT,
					servlet, request, response, e);
			throw e;
		} catch (RuntimeException e) {
			support.fireInstanceEvent(InstanceEvent.AFTER_SERVICE_EVENT,
					servlet, request, response, e);
			throw e;
		} catch (Throwable e) {
			support.fireInstanceEvent(InstanceEvent.AFTER_SERVICE_EVENT,
					servlet, request, response, e);
			throw new ServletException(sm.getString("filterChain.servlet"), e);
		} finally {
			//安全校验
			if (Globals.STRICT_SERVLET_COMPLIANCE) {
				lastServicedRequest.set(null);
				lastServicedResponse.set(null);
			}
		}

	}

```

在internalDoFilter(ServletRequest, ServletResponse)方法中，可以看到servlet是在所以FilterChain最一个Filter前置过滤完成后调用，调用完成后依次递归出来，逆序调用FilterChain上的Filter后置过滤servlet。在Filter中可以添加校验逻辑，日志记录等信息。到目前为止，tomcat已经完成了使用servlet处理请求信息的功能。

FilterChain的模式是设计模式中的责任链模式，该模式的必须要经过所有的链上过滤完后才会执行最后的操作。






