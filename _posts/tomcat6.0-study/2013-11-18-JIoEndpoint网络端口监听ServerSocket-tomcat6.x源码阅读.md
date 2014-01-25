---
layout: post
title: JIoEndpoint网络端口监听ServerSocket-tomcat6.x源码阅读
date: 2013-11-18
categories: java
---
**2013-11-18**

tomcate需要不间断监听网络端口，接收客户端请求，完成响应。JIoEndpoint的角色是监听网络端口请求，然后见接收到的网络请求转给servlet处理。他负责管理ServerSocket(TCP/IP),接收socket,交由工作线程来处理.在JIoEndpoint类内部规范了socket的处理接口，使用线程池处理每一个socket.

JIoEndpoint使用ServerSocket接收来客户端的请求socket,收到请求后从线程池中去一个工程线程来处理socket.JIoEndpoint是如何完成呢?通过ServerSocketFactory创建ServerSocket，多个Acceptor线程类不间断监听请求，将请求分配给Worker，Worker调用Handler.process()完成处理过程.下面看看这几个组件的详细信息.

*Acceptor*  
ServerSocket接收socket请求线程类，也即使用ServerSocket监听网络端口请求。是线程类，在tomcate中会同时运行多个线程实来提高监听效率和成功率，服务的时效性和可靠性都有提升，避免在一个线程挂掉的情况下无法监听客户端请求。类只有一个方法`run()`,代码如下：

```java

	/**
	 * Server socket acceptor thread.
	 * 服务端socket监听请求接收线程类
	 */
	protected class Acceptor implements Runnable {

		/**
		 * The background thread that listens for incoming TCP/IP connections
		 * and hands them off to an appropriate processor.
		 * 后台线程不间断监测来自客户的TCP/IP连接请求任务，将请求分配给一个工作线程来处理
		 */
		public void run() {

			// Loop until we receive a shutdown command
			//判断是否需要监听
			while (running) {

				// Loop if endpoint is paused
				//暂停监听，每1秒查看一次是否撤销暂停监听
				while (paused) {
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						// Ignore
					}
				}

				// Accept the next incoming connection from the server socket
				try {
					//接收来自客户发的请求连接
					Socket socket = serverSocketFactory
							.acceptSocket(serverSocket);
					serverSocketFactory.initSocket(socket);
					// Hand this socket off to an appropriate processor
					//处理socket
					if (!processSocket(socket)) {
						// Close socket right away
						try {
							//关闭请求连接
							socket.close();
						} catch (IOException e) {
							// Ignore
						}
					}
				} catch (IOException x) {
					if (running)
						log.error(sm.getString("endpoint.accept.fail"), x);
				} catch (Throwable t) {
					log.error(sm.getString("endpoint.accept.fail"), t);
				}
				// The processor will recycle itself when it finishes
			}
		}
	}

```
从上面的代码中可以看出，它的任务很简单：接收客户端请求连接，为socket分配工作线程处理。


*processSocket(Socket)*  
负责处理socket，为socket选择一个工作线程.如果tomcate中使用了线程运行池的话，则从运行池中取一个线程运行器来处理socket，否则从工作线程池中取一个工作线程来处理。(没弄明白区别是啥，估计跟性能有关)

*SocketProcessor*  
是socket的处理线程类，在线程方法run()中处理从Acceptor中传递过来的socket，它本身不做任何逻辑处理，只负责调用handler对socket的处理，详细代码如下.

```java

	/**
	 * This class is the equivalent of the Worker, but will simply use in an
	 * external Executor thread pool.
	 */
	protected class SocketProcessor implements Runnable {

		protected Socket socket = null;

		public SocketProcessor(Socket socket) {
			this.socket = socket;
		}

		public void run() {

			// Process the request from this socket
			if (!setSocketOptions(socket) || !handler.process(socket)) {
				// Close socket
				try {
					socket.close();
				} catch (IOException e) {
				}
			}

			// Finish up this request
			socket = null;

		}

	}
```
在SocketProcessor的run()方法中先对socket做配置操作，然后调用Handler.process(Socket)处理.Handler是tomcate定义处理Socket的接口.

*Handler*  
该接口时JIoEndpoint定义来处理socket的接口，具体的协议实现具体方法。HTTP1.1协议处理类Http11Protocol中的内部类Http11ConnectionHandler实现了本接口，处理关于HTTP1.1协议socket.

tomcate是多线程处理socket，Worker是JIoEndpoint中定义socket处理工作线程运行器，一个socket分配一个Worker处理，代替如果在tomcate的配置文件中server.xml没有配置实用线程组池来作为socket的工作线程.

*Worker*  
负责处理socket的工作线程，Worker实现了socket的等待队列功能。当Acceptor与客户端建立连接产生socket时，将从工作线程组中取出一个工程线程处理socket，如果当前工作线程不可用,则同步等待工作进程到可用状态socket才会被处理.Worker有3个属性:

+ Thread:工作线程所依附的线程实体运行
+ available:用于标记当前工作线程是否可用
+ Socket:工作线程所要处理的socket

Worker中3个重要方法

`assign(Socket)`给Worker设置要处理的Socket.是线程同步方法，可能存在多个线程同时获取个一个Worker而造成socket丢失的情况。在方法内部判断Worker是否可用，如果不可用，则循环等待可用状态.

```java

		/**
		 * Process an incoming TCP/IP connection on the specified socket. Any
		 * exception that occurs during processing must be logged and swallowed.
		 * <b>NOTE</b>: This method is called from our Connector's thread. We
		 * must assign it to our own thread so that multiple simultaneous
		 * requests can be handled.
		 * 
		 * @param socket
		 *            TCP socket to process
		 */
		synchronized void assign(Socket socket) {

			// Wait for the Processor to get the previous Socket
			while (available) {//判断Worker是否被占用
				try {
					wait();//进入wait等待
				} catch (InterruptedException e) {
				}
			}

			// Store the newly available Socket and notify our thread
			this.socket = socket;//设置socket
			available = true;//设置线程使用中
			notifyAll();//唤醒所有sleep的线程

		}
```

`await()`







