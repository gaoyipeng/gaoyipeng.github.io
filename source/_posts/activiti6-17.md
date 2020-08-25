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

除此之外**错误结束事件**抛出的错误编码还可以被**错误边界事件**捕获。这个在前面的子流程章节已经见到过，流程图如下：

![image-20200825094750481](/images/activiti6-17/image-20200825094750481.png)

介绍子流程时我们我们没有演示**错误边界事件**捕获，此处实际操作一遍。前面的步骤就不一一演示了，直接演示财务审批（不同意）。

这里贴出异常结束事件的XML:

```xml
<endEvent id="errorendevent2" name="End">
    <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
</endEvent>
```

错误边界事件XML：

```xml
<boundaryEvent id="boundaryerror1" name="Error" attachedToRef="subprocessPay">
    <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
</boundaryEvent>
```



![image-20200825142333297](/images/activiti6-17/image-20200825142333297.png)

![image-20200825143225996](/images/activiti6-17/image-20200825143225996.png)

此时我们再次查看流程图，可以看到异常结束事件抛出异常后，被**错误边界事件**捕获，流程流转至跳转申请节点：

![image-20200825143341686](/images/activiti6-17/image-20200825143341686.png)



## 1.3 终止结束事件

当流程执行到终止结束事件，当前的流程将会被终结，终止结束事件可以使用在嵌入子流程、调用子流程、事件子流程或者事务子流程中。

一般情况下，当存在多实例的调用过程或嵌入式子流程时，只会终止一个实例，其他的实例与流程实例不会受影响。例如我们前面的子流程实例，如果子流程结束，主流程是不会一起结束的。

终止结束事件使用terminateEventDefinition元素作为事件定义，如果将该元素的activiti:terminateAll属性设置为true，那么当终止结束事件被触发事时，流程实例的全部执行流均会被终结。

XML表示：

```xml
<endEvent id="myEndEvent >
  <terminateEventDefinition activiti:terminateAll="true"></terminateEventDefinition>
</endEvent>
```

