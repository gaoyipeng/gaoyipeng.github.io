---
title: Activiti（17）Activiti启动事件
date: 2020-08-21 15:38:58
tags: Activiti
categories: Activiti
description: Activiti结束事件
typora-root-url: ..
password: kiki
---

本节我们介绍Activiti的结束事件类型。

# 1、结束事件

结束事件标志着（子）流程的（分支的）结束。结束事件**总是抛出（型）事件**。这意味着当流程执行到达结束事件时，会抛出一个*结果*。结束事件一共有以下几种：

![image-20200821155350294](/images/activiti6-17/image-20200821155350294.png)

## 1.1 空结束事件

 “空”结束事件，意味着当到达这个事件时，抛出的*结果*没有特别指定。因此，引擎除了结束当前执行分支之外，不会多做任何事情。这个大家很好理解，不做演示了。

XML表示：

```xml
<endEvent id="end" name="endEvent" />
```

## 1.2 错误结束事件

当流程执行到达**错误结束事件**时，结束执行的当前分支，并抛出错误。

XML表示：

错误结束事件，表示为结束事件，加上*errorEventDefinition*子元素：

```xml
<endEvent id="myErrorEndEvent">
  <errorEventDefinition errorRef="myError" />
</endEvent>
```

错误开始事件，往往和错误结束事件一起出现。错误结束事件抛出一个异常编码，只要和异常启动事件设置的异常编码匹配，即可开启事件子流程。在前面的事件子流程章节，我们已经见过了，这里就不再赘述：

![image-20200820104351195](/images/activiti6-17/image-20200820104351195.png)

除此之外**错误结束事件**抛出的错误编码还可以被错误边界事件捕获。这个我们学习边界事件时再介绍。

## 1.3 终止结束事件