---
layout: post
title:  "前端学习之xml dom 系列之节点操作(节点属性和Attr)"
date:   2013-09-08 21:06:09
categories: jekyll update
---
>[w3school xml dom](http://www.w3school.com.cn/xmldom/index.asp) 

>xml dom 中标签也即节点，不同位置节点的含义是不一样的

1.节点类型

- Document
  >表示整个文档(DOM树的根节点)
  * Element(max.one)
  * Processinglnstruction
  * Comment
  * DocumentType

- DocumentFragment
  >表示经量级的Document对象，其中容纳量一部分文档
  * Processsinglnstruction
  * Comment
  * Text
  * CDATASection
  * EntityReference

- DocumentType
  > 向文档定义的实体提供接口
  * None

- Processinglnstruction
  > 表示处理指令
  * None

- EntityReference
  > 表示element元素
  * Text
  * Comment
  * Processinglnstruction
  * CDATASection
  * EntityReference

- Attr
  > 表示属性
  * Text
  * EntityReference

- Text
  > 表示元素或者属性中的文本内容
  * None

- CDATASection
  > 表示文档中的CDATA区段(文本不会被解析器解析)
  * None

- Comment
  > 表示注释
  * None

- Entity
  > 表示实体
  * Processinglnstruction
  * Comment
  * Text
  * CDATASection
  * EntityReference类型

- Notation
  > 表示在DTD中声明的符号
  * None

2.节点类型 - 所返回的值
 
- Document	#document	null
- DocumentFragment	#documentfragment	null
- DocumentType	#doctype名称	null
- EntityReference	实体引用名称	null
- Element	element name	null
- Attr	属性名称	属性值
- Processinglnstruction	target	节点的内容
- Comment	#coment	注释文本
- Text	#text	节点内容
- CDATASection	#data-section	节点内容
- Entity	实体名称	null
- Notation	符号名称	null

Attr对象
> Element的属性，也是节点的一种，所以继承了Node对象的属性和方法，注意：属性没有父节点，也不是元素的子节点

Attr对象的属性

* baseURI:返回属性的绝对基本URI
* isId:返回判断是否为ID属性类型
* localName:返回属性名称的本地部分
* name:返回属性的名称
* namespaceURL:返回命名空间URI
* nodeName:返回节点的名称，依据其类型
* nodeType:返回节点类型
* nodeValue:设置或者返回节点的值，依据其类型
* ownerDocument:返回属性所属的跟元素(document对象)
* onwerElement:返回属性所附属的元素节点
* prefix:设置或返回属性的命名空间前缀
* schemaTypeInfo:返回与属性相关联的类型信息
* specified:如果属性值被设置在文档中，写返回true,如果其默认值被设置在DTD/Schema中，则返回false
* textContent:设置或返回属性的文本内容
* value:设置或者返回属性的值
