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

