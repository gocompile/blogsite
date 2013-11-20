#前端学习之xml dom 系列之节点操作(Document-DOMParser-Element)
>[w3school xml dom](http://www.w3school.com.cn/xmldom/index.asp) 

1.Comment 对象(略过)

2.Document 对象
> 是一个文档树的根，提供对文档数据的最初的访问入口.

Document 属性

* async
  > 规定XML文档的下载是否应该被同步处理。

* childNodes
  > 返回属于的子节点的节点列表。

* doctype
  > 返回与文档相关的文档类型声明(DTD)。

* documentElement
  > 返回文档的根节点。

* documentURI
  > 设置或者返回文档的位置。

* domConfig
  > 返回normailzeDocument()被调用时所使用的配置。

* firstChild
  > 返回文档的首个子节点。

* implementation
  >返回处理该文档的DOMImplemenation对象。

* inputEncoding
  > 返回用于文档的编码方式。

* lastChild
  > 返回文档的最后一个节点。

* nodeName
  > 依据节点的类型返回其名称。

* nodeType
  > 返回节点的节点类型。

 * nodeValue
  > 根据节点的类型来设置或者返回节点的值。

* strictErrorChecking
  > 设置或返回是否强制进行错误检查。

* xmlEncoding
  > 返回文档的编码方法。

* xmlStandalone
  > 设置或返回文档是否为standalone

* xmlValue
  > 设置或返回文档的XML版本。

Document 方法

* adopNode(sourcenode)
  > 从另一个文档想本文档选定一个节点，然后返回被选节点。

* createAttribute(name)
  > 创建拥有制定名称的属性节点，并返回新的Attr对象。

* createAttributeNS(uri,name)
  > 创建拥有制定名称和命名空间的属性节点，并返回新的Attr对象。

* createCDATASection(data)
  > 创建CDATA区段节点。

* createComment(data)
  > 创建注释节点并返回Comment对象。

* createDocumentFragment()
  > 创建空的DocumentFragment对象，并返回该对象。

* createElement(name)
  > 创建元素节点。

* createElementNS(ns,name)
  > 创建带有指定命名空间的元素节点。

* createEvent(eventType)
  > 创建新的Event对象。

* createEntityReference(name)
  > 创建EntityReference对象，并返回此对象。

* createExpression(xpathText,namespaceURLMappder)
  > 创建一个XPath表达式以供稍后计算。

* createProcessinglnstruction(target,data)
  > 创建Processinglnstruction对象并返回。

* createRange()
  > 创建Range对象并返回。

* evaluate(xpathText,contextNode,namespaceURLMappder,resultType,result)
  > 计算一个Xpath表达式。

* createTextNode(data)
  > 创建文本节点。

* getElemetById(elementid)
  > 查找具有指定的唯一ID的元素。

* getElementByTagName(name)
  > 返回所具有指定名称的元素节点。

* getElementByTagNameNS(ns,name)
  > 返回所具有指定名称和命名空间的元素节点。

* importNode(importNode,deep)
  > 把一个节点从另一个文档复制到该文档以便应用。

* loadXML(xml)
  > 通过解析XML标签字符串来组成文档。

* normalizeDocument()
  > fuck。

* renameNode(node,uri,name)
  > 重命名元素或者属性节点。


3.DocumentType 对象(略过)

4.DomImplementation 对象(略过)

5.DOMPareser 对象
> 解析XML标记来创建一个文档

* 构造函数 DOMPareser()

DOMPareser 方法
* pareserFromString(text,contentType)
  > 解析XMl标记创建文档。

6.Element
> 表示文档中的元素，可包含属性、其他元素或者文本。文本在文本节点中。

Element 属性

* attributes
  > 返回元素属性的NamedNodeMap。

* baseURI
  > 返回元素的绝对基准的URI。

* childNodes
  > 返回元素的子节点的NodeList。

* firstChild
  > 元素的首个子节点。

* lastChild
  > 元素的最后一个子节点。

* localName
  > 元素名称的本地部分。

* namespaceURI
  > 元素的命名空间的URI。

* nextSibling
  >元素之海紧跟的节点。

* nodeName
  > 已经节点类型返回节点名称。

* nodeType
  > 节点类型。

* ownerDocument
  >元素所属的根元素。

* parentNode
  > 元素的父节点。

* prefix
  > 元素命名空间前缀。

* previousSibling
  > 元素之前紧随的节点。

* tagName
  > 元素名称。

* textContent
  > 元素以其后代的文本内容。

Element 方法

* appendChild(node)
  > 向节点的子节点列表末尾添加新的子节点。

* cloneNode(include_all)
  > 克隆节点。

* compareDocumentPosition(node)
  > 比较两节点的文档位置。

* dispatchEvent(evt)
  > 给节点分派一个合成事件。

* getAttrinute(name)
  > 属性的值。

* getAttributeNS(ns,name)
  > 根据命名空间返回属性的值。

* getAttributeNode(name)
  > 以Attribute对象返回属性节点。

* getAttributeNS(ns,name)
  > 找到具有指定标签和命名空间的元素。

* hasAttribute(name)
  > 是否拥有指定属性。

* hasAttributeNS(ns,name)
  > 是否拥有指定属性。

* hasAttributes()
  > 是否拥有属性。

* hasChildNodes()
  > 是否拥有子节点。

* insertBefore(new_node,existing_node);
  > 在某个子节点之前插入新子节点。

* isEqualNode(node)
  > 两个节点是否相等。

* isSameNode(node)
  > 是否为同一个节点。

* isSupported(feature,version)
  > 是支持特性。

* lookupNamespaceURI(prefix)
  > 匹配指定前缀的命名空间URI。

* lookuoPrefix(URI)
  > 匹配指定命名空间URI的前缀。

* removeAttribute(name)
  > 移除属性。

* removeAttributeNS(ns,name)
  > 移除指定命名空间的属性。

* reemoveAttributeNode(node)
  > 移除属性节点。

* removeChild(node)
  > 移除子节点。

* replaceChild(new_node,old_node)
  > 替换子节点。

* setAttribute(name,value)
  > 添加新属性。

* setAttributeNS(ns,name,value)
  > 添加新属性。

  @date:2013-04-13 20:39 @author:gocompile