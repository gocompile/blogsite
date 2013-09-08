---
layout: post
title:  "前端学习之xml dom 系列之节点操作(Event)"
date:   2013-09-08 21:06:09
categories: jekyll update
---
#前端学习之xml dom 系列之节点操作(Event)
>[w3school xml dom](http://www.w3school.com.cn/xmldom/index.asp) 

1.Event
> Event对象属性提供了有关事件的细节，Event对象的方法可以控制事件的传播。Event对象传递给事件句柄函数。

标准Event 属性

* bubbles
  > 返回布尔值，指示事件是否是起泡类型。

  > 在2级DOM中，事件传播分为三个阶段。

    >> 第一，捕获阶段，事件从Document对象沿着文档树向下传递给目标节点。如果目标的任何一个先辈专门注册了捕获事件句柄，那么事件传播过程中运行这些句柄。

    >> 第二发生在目标节点自身。直接注册砸目标上的合适的事件句柄将运行。这与0级事件模型提供的事件处理方法相似。

    >> 第三，起泡阶段，事件将从目标元素向上传播会或者起泡回Document对象的文档层次。

* cancelable
  > 返回布尔值，指示事件是否拥有可取消的默认动作。

* currentTarget
  > 返回其事件监听器触发该事件的元素。

* eventPhase
  > 返回事件传播的当前阶段。

* target
  > 返回触发此事件的元素(事件的目标节点)。

* timeStamp
  > 返回事件生成的日期和时间。

* type
  > 返回当前Event对象表示的事件的名称。

标准Event 方法

* initEvent(eventType,canBbule,cancelable)
  > 初始化新创建的Event对象的属性。

* preventDefault()
  > 通知浏览器不用不要执行与事件关联的默认动作。

* stopPropagation()
  > 终止事件在传播过程的捕获，目标处理或者起泡阶段进一步传播。调用该方法后，该节点上处理该事件的处理程序将被调用，事件不再被分派到其他节点。

2.HTMLCollection
> 是一个接口表示HTML元素的集合，提供可以遍历列表分方法和属性。
HTMLCollection 属性

* cssRules
  > 只读属性，返回只是列表长度的整数。

HTMLCollection 方法

* item(index)
  > 返回指定位置的元素

* namedItem(name)
  > 返回集合中name属性或具有id属性具有指定值的元素(节点)。


3.HTMLDocument
> HTMLDocument接口提供了对HTML层级的访问。

HTMLDocument 接口对DOM Document接口进行了扩展，定义了HTML专用的属性和方法。


4.HTMLElement
> 一个HTML文档中的每个元素都有和元素的属性对应的属性。

HTMLElement 属性

* className
  > 规定元素的class属性。

* dir
  > 规定元素的dir属性，声明了文档文本的方向。

* id
  > 规定元素的id属性，在一个文档中具有唯一性。

* innerHTML
  > 规定了元素所包含的字符串，不包括元素自身的开始标记和结束标记，查询这个属性会将元素的内容作为一个HTMl文本串返回。

* lang
  > 规定元素的lang属性，声明了元素内容的语言代码。

* offsetHeight，offsetWidth
  > 返回元素的高度和宽度，以像素为单位。

* offsetLeft
  > 返回当前元素的左边界到他的包含元素的左边界的偏移量，以像素为单位。

* offsetTop
  > 返回当前元素的上边界到它包含元素的上边界的偏移量，以像素为单位。


* offsetParent
  > 返回对最近的动态定位的包含元素的引用，所有的偏移量都根据该元素来决定。

* scrollHeiht,scrollWidth
  > 返回元素的完整高度和宽度，以像素为单位。

* scrollTop,scrollLeft
  > 设置或返回已经滚动到元素的左边界或上边界的像素数。

* style
  > 返回为当前元素设置内联CSS样式的style属性的值。

* title
  > 规定元素的title属性，鼠标悬浮在元素上时显示。


  
  @date:2012-04-14 02:10
