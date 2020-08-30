---
title: Activiti（17）Activiti结束事件
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

**错误结束事件**，往往和**错误开始事件**一起出现。**错误结束事件**抛出一个异常编码，只要和**错误启动事件**设置的异常编码匹配，即可开启事件子流程。在前面的事件子流程章节，我们已经见过了，这里就不再赘述：

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

*attachedToRef="subprocessPay"*表示这个错误边界事件的监听对象是这个id为subprocessPay的子流程。

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

我们画一个简单流程来验证一下。流程启动后有一个并行网关，分别是一个`子流程`、一个`人事审批`节点。终止结束事件勾选了终止全部。我们先走完子流程，看看人事这条线是否会随之结束。

![image-20200826112351885](/images/activiti6-17/image-20200826112351885.png)

流程定义如下：

```xml
<!-- -->
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <process id="terminateEndEvent" name="终止结束事件" isExecutable="true">
    <startEvent id="startevent1" activiti:initiator="applyUserId"></startEvent>
    <subProcess id="sid-050C0CDB-2A9B-4C97-993B-6E5A194537D0" name="subProcess">
      <startEvent id="sid-3EFEFC9A-02C4-447F-8359-42D132E8CB5A"></startEvent>
      <userTask id="manageApprove" name="部门领导审批" activiti:candidateUsers="leaderuser"></userTask>
      <endEvent id="hruser">
        <terminateEventDefinition activiti:terminateAll="true"></terminateEventDefinition>
      </endEvent>
      <sequenceFlow id="sid-DE7A6A1E-0C9D-45D3-849E-D38DA6AEE574" sourceRef="sid-3EFEFC9A-02C4-447F-8359-42D132E8CB5A" targetRef="manageApprove"></sequenceFlow>
      <sequenceFlow id="sid-B0BD8E26-9C66-46D8-9468-234AFBB8CCE0" sourceRef="manageApprove" targetRef="hruser"></sequenceFlow>
    </subProcess>
    <userTask id="hrApprove" name="人事审批" activiti:candidateUsers="hruser"></userTask>
    <endEvent id="sid-C6D99B13-C7EF-4D71-8A93-7B086ECAD8CA"></endEvent>
    <sequenceFlow id="sid-C5FC93E8-D838-4F4E-B076-EDC38353FDE5" sourceRef="hrApprove" targetRef="sid-C6D99B13-C7EF-4D71-8A93-7B086ECAD8CA"></sequenceFlow>
    <parallelGateway id="sid-485D8B29-FC51-479A-B33D-4182C26B1BB6"></parallelGateway>
    <sequenceFlow id="sid-44C4A09F-4671-47BA-88A4-14D516FFCD71" sourceRef="startevent1" targetRef="sid-485D8B29-FC51-479A-B33D-4182C26B1BB6"></sequenceFlow>
    <sequenceFlow id="sid-572C0962-598F-45C4-9363-B5A00E276C42" sourceRef="sid-485D8B29-FC51-479A-B33D-4182C26B1BB6" targetRef="hrApprove"></sequenceFlow>
    <sequenceFlow id="sid-67524AB4-99E0-4A10-8ACB-6D69C279A6F2" sourceRef="sid-485D8B29-FC51-479A-B33D-4182C26B1BB6" targetRef="sid-050C0CDB-2A9B-4C97-993B-6E5A194537D0"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_terminateEndEvent">
    //连线信息省略
  </bpmndi:BPMNDiagram>
</definitions>
```

启动流程：

![image-20200826112714350](/images/activiti6-17/image-20200826112714350.png)

![image-20200826112730272](/images/activiti6-17/image-20200826112730272.png)

我们在部门领导审批节点、人事审批节点分别指定了审批人：`leaderuser`，`hruser`。我们先来查看一下他们的待办：

![image-20200826113449516](/images/activiti6-17/image-20200826113449516.png)

![image-20200826113528952](/images/activiti6-17/image-20200826113528952.png)

接下来部门领导审批通过：

![image-20200826113930104](/images/activiti6-17/image-20200826113930104.png)

再次查看流程图，发现整个流程都已关闭：

![image-20200826114038834](/images/activiti6-17/image-20200826114038834.png)

而且`hruser`已经没有对应的待办了，符合我们的预期。

![image-20200826114159645](/images/activiti6-17/image-20200826114159645.png)

## 1.4 取消结束事件

取消结束事件只能与BPMN事务子流程结合使用。 当到达取消结束事件时，会抛出取消事件，它必须被取消边界事件捕获。 取消边界事件会取消事务，并触发补偿机制。

XML :

```xml
<endEvent id="myCancelEndEvent">
  <cancelEventDefinition />
</endEvent>
```

这个我们后面学习事务子流程时再介绍。