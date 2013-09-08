---
layout: post
title:  "前端学习之xml dom 系列之节点操作(CSS)"
date:   2013-09-08 21:06:09
categories: jekyll update
---
#前端学习之xml dom 系列之节点操作(CSS)
>[w3school xml dom](http://www.w3school.com.cn/xmldom/index.asp) 

CSS2Properties对象
> 表示一组CSS样式属性及其值，为每一个CSS规范定义的每一个CSS属性都定义一个javaScript属性。
> 一个HTMLElement的style属性是一个可读可写的CSS2Properties对象，但Window.getComputeStyle()中返回的CSS2properties对象属性是只读。

CSS2Properties 属性

* cssText 属性
  > 是一组样式属性及其值的文本表示。

CSSRule 对象
> 是一个基类，用于定义CSS样式表中的任何规则，包括规则集(rule sets)和@规则(at-rules)。

CSSRuls 属性(属性都是只读)

* cssText
  > 返回规则的文本表示。

* parentRule
  > 返回包含规则。

* parentStyleSheet
  > 返回该规则所属的stylesheet对象。

* type
  > 规则类型。

CSSStyleRule 对象
> CSSSyle 对象表示CSS样式中一个单独的规则集。

CSSStyleRule 属性

* selectorText
  > 设置或者返回该规则中选择器的文本表示。

* style
  > 返回CSSStyleDeclaration对象，即CSS选择器的样式值。

CSSStyleSheet 对象
> 表示一个单独的CSS样式表。CSS样式表由CSS规则组成，可以通过CSSRule对象操作每条规则，CSSStyleSheet对象允许查询、插入和删除样式表规则。

CSSStyleSheet 属性

* cssRules
  > 以数组的形式返回样式表中的所有CSS规则。

* disabled
  > 该属性指示是否已应用当前样式表。

* href
  > 返回样式表的位置(URL)，如果是内联样式表，则为null。

* media
  > 规则样式信息预期的目标媒介。

* ownerNode
  > 返回将该样式表与稳定相关联的节点。

* ownerRule
  > 如归哦该样式表来自@import规则，ownerRule属性将包含CSSImportRule。

* parentStyleSheet
  > 返回包含该样式表的样式表。

* title
  > 返回当前样式表的标题。

* type 
  > 规定该样式表的样式表语言。

CSSStyleRule 方法

* deleteRule(index)
  > 从指定位置删除规则的DOM标准方法。

* insertRule(rule,index)
  > 向样式表中插入一条新规则的DOM标准方法。

@date:2013-04-13 11:42 @author:gocompile
