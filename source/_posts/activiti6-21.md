---
title: Activiti（21） 网关
date: 2020-09-07 16:16:23
tags: Activiti
categories: Activiti
description: Activiti（21） 网关
typora-root-url: ..
password: kiki
---

网关用来控制流程的流向。网关一共有4种：

![image-20200907164855419](/images/activiti6-21/image-20200907164855419.png)

## 1、互斥网关（排他网关）

排他网关， 用来在流程中实现**决策**。当流程执行到这个网关，所有外出顺序流都会被处理一遍。 其中条件解析为true的顺序流（或者没有设置条件，概念上在顺序流上定义了一个*'true'*） 会被选中，让流程继续运行。

注意：**如果多个顺序流的条件结果为true， 那么XML中的第一个顺序流（也只有这一条）会被选中，并用来继续运行流程。 如果没有选中任何顺序流，会抛出一个异常。**

### 1.1 条件表达式

顺序流上可以设置条件表达式（**UEL**），使用的表达式需要返回boolean值，否则会在解析表达式时抛出异常。条件表达式分为2种：**值表达式**和**方法表达式**。

```xml
<!--值表达式:order为流程变量-->
<conditionExpression xsi:type="tFormalExpression">
  <![CDATA[${order.price > 100 && order.price < 250}]]>
</conditionExpression>
<!--方法表达式:调用方法返回一个boolean值-->
<conditionExpression xsi:type="tFormalExpression">
  <![CDATA[${order.isStandardOrder()}]]>
</conditionExpression>
```

### 1.2 默认顺序流

所有的BPMN 2.0任务和网关都可以设置一个并且只能设置一个**默认顺序流**。 默认顺序流的条件设置不会生效。

默认顺序流显示为了普通顺序流，起点有一个“斜线”标记。

![image-20200907173727499](/images/activiti6-21/image-20200907173727499.png)

## 2、并行网关