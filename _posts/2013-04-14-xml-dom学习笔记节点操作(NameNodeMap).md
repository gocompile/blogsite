---
layout: post
title:  "前端学习之xml dom 系列之节点操作(NameNodeMap)"
date:   2013-09-08 21:06:09
categories: jekyll update
---
>[w3school xml dom](http://www.w3school.com.cn/xmldom/index.asp) 

1.NameNodeMap
> 通过节点名称来访问NameNodeMap中的节点。

NameNodeMap 属性

* length
  > 列表中节点的数目。

NameNodeMap 方法

* getNamedItem(name)
  > 返回指定节点。

* getNamedItemNS(ns,name)
  > 返回指定节点。

* item(index)
  > 返回指定节点。

* removeNamedItem(name)
  > 移除指定节点

* removeNamedItemNS(ns,name)
  > 移除指定节点。

* setNamedItem(name)
  > 设置指定的节点。

* setNamedItemNS(ns,name)
  > 设置指定节点。

2.Node
> Node对象时整个DOM的主要数据类型，代表文档树中的一个单独节点，

Node 属性

* 属性与Element属性一样。

Node 方法

* 方法与Element方法一样。

3.NodeList
> Node列表集

NodeList 属性

* length
  > 列表长度。

NodeList 方法

* item(index)
  > 返回指定索引节点。


3.Parser Errors(略过)

4.Processinglnstruction (略过)

5.Range (略过)

6.RangeException(略过)

7.Text 
> 表示HTML或XML文档中的一系列纯文本。继承自CharacterData接口，通过CharacterData接口继承的data属性和从Node接口继续的nodeValue属性，可以访问Text节点的文本内容。

Text 属性

* data
  > 元素或者属性文本。

* isElementContentWhitespace
  > 判断文本节点是否包含空白字符。

* length
  > 文本长度。

* wholeText
  > 以文档中的顺序向此节点相邻文件节点的所有文本。

Text 方法

* appendDate(string)
  > 追加文本。

* deleteData(start,length)
  > 删除文本。


* insertDate(start,string)
  > 插入文本。

* replaceData(start,length,string)
  > 替换数据。

* replaceWholeText(string)
  > 替换整个节点的数据。

* spiltText(offset)
  > 分割文本

* substringData(start,length)
  > 返回子字符串。



8.XMLHttpRequest
> 提供对HTTP协议的完全访问，包括POST和HEAD请求以及普通的GET请求的能力。XMLHttpRequest可以同步或异步返回web服务器的响应，并且能够以文本或者一个DOM文档的形式返回内容。

XMLHttpRequset 属性

* readyStat
  > HTTP请求的状态。
    >> 0	Uninitalized	初始化状态，XMLHttpRequest对象创建或被abort()方法重置。
    >> 1	Open	open()方法已调用，但是send()方法未被调用，请求还没有被发送。
    >> 2	send	send()方法已调用，HTTP请求已被发送到Web服务器。未接收到响应。
    >> 3	Receiving	所有响应头部都已经接收到，响应体开始接收但未完成。
    >> 4	Loaded	HTTP响应已经完全接收。
  > readyStat 的值不会递减，除非当一个请求在处理过程中的时候调用了abort()或open()方法，每次每个属性的值增加的时候，都会触发onreadystatchange事件句柄。

* responseText
  > 从web服务器接收到的响应体，不含头部，如果还没有收到数据时，为空串。

* responseXML
  > 对请求的响应，解析为XML并作为Document对象返回。

* status
  > 服务器返回的HTTP状态码。如果readyStat小于3时，读取该属性会引发异常。

XMLHttpRequest 事件句柄

* onreadystatechange
  > 每次readyState属性改变的时候调用的事件句柄函数。当readyState=3时，可能被调用多次。

XMLHttpRequest 方法

* abort()
  > 取消当前响应，关闭连接且结束任何未决的网络活动。将readyState置为0状态。

* getAllResponseHeaders()
  > 把HTTP响应头部作为解析的字符串返回。

* getResponseHeader(string)
  > 返回指定的HTTP响应头部的值，参数是HTTP响应头部的名称，忽略大小写。

* open()
  > 初始化HTTP请求。

* send(data)
  > 发送HTTP请求，使用传递给open()方法的函数，以及传递给该方法的可选请求体。

* setRequestHeader(header...)
  > 向一个打开但未发送的请求设置或者添加一个HTTP请求。


XMLHTTPRequest.open(method,url,async,username,password)
  > 初始化HTTP请求参数。


@date:2013-04-14 21:29 @author:gocompile



