---
title: Activiti（19） 边界事件
date: 2020-08-30 11:33:03
tags: Activiti
categories: Activiti
description: Activiti（19） 边界事件
typora-root-url: ..
password: kiki
---

边界事件都是*捕获*事件，它会附在一个环节上。当节点运行时， 事件会*监听*对应的触发类型。

![image-20200830163241054](/images/activiti6-19/image-20200830163241054.png)

## 1、边界错误事件

全称：节点边界上的中间*捕获*错误事件，它会捕获节点范围内抛出的错误。

边界错误事件，大多用于**子流程**， 或**调用活动**。往往和**错误结束事件**一起使用。

当捕获了错误事件时，边界任务绑定的节点就会销毁， 也会销毁内部所有的执行分支 （比如，同步节点，内嵌子流程，等等）。 流程执行会继续沿着边界事件的外出连线继续执行。

这个在介绍 **结束事件—错误结束事件** 时已经演示过了，这里不再赘述。



## 2、定时边界事件

定时边界事件就是一个暂停等待警告的时钟。当流程执行到绑定了边界事件的环节， 会启动一个定时器。默认当定时器触发时（比如，一定时间之后），环节就会中断， 并沿着定时边界事件的外出连线继续执行。但是如果不想让原环节中断，可以将*cancelActivity*属性设置为**false**。

*cancelActivity*：

- true：中断  ，边界事件2个圆圈是实线。
- false：非中断，边界事件2个圆圈是虚线线。

### 2.1 定义流程

我们此处定义一个简单流程来验证一下：

部门经理节点设置了一个定时边界事件，设置为5分钟后。并且取消活动打钩（cancelActivity=true）。我们看看流程到达部门经理审批节点5分钟后，会有什么变化。

![image-20200830215139310](/images/activiti6-19/image-20200830215139310.png)

![image-20200831141828981](/images/activiti6-19/image-20200831141828981.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <process id="timedBoundaryEvent" name="定时边界事件" isExecutable="true">
    <documentation>定时边界事件</documentation>
    <startEvent id="timedBoundaryStart" name="定时边界事件" activiti:initiator="applyUserId"></startEvent>
    <userTask id="deptApprove" name="部门经理审批" activiti:candidateGroups="deptLeader"></userTask>
    <userTask id="hrApprove" name="人事审批" activiti:candidateGroups="hr"></userTask>
    <sequenceFlow id="sid-7601B3F7-54E1-46F0-A829-3594101FC3DD" sourceRef="deptApprove" targetRef="hrApprove"></sequenceFlow>
    <userTask id="manageApprove" name="manager审批" activiti:candidateGroups="generalManager"></userTask>
    <boundaryEvent id="timedBoundaryId" attachedToRef="deptApprove" cancelActivity="true">
      <timerEventDefinition>
        <timeDuration>PT5M</timeDuration>
      </timerEventDefinition>
    </boundaryEvent>
    <endEvent id="sid-D9E71CF8-CABF-445A-957C-024AEB9D7F51"></endEvent>
    <sequenceFlow id="sid-F621A86A-13E3-4D59-82CB-4670F6331C10" sourceRef="hrApprove" targetRef="sid-D9E71CF8-CABF-445A-957C-024AEB9D7F51"></sequenceFlow>
    <sequenceFlow id="sid-F6F7372A-2378-41E6-8712-4A602A923B22" sourceRef="timedBoundaryStart" targetRef="deptApprove"></sequenceFlow>
    <sequenceFlow id="sid-4DB7A535-3C11-4931-87DA-4DB6037D0A90" name="触发定时边界事件" sourceRef="timedBoundaryId" targetRef="manageApprove"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_timedBoundaryEvent">
    <bpmndi:BPMNPlane bpmnElement="timedBoundaryEvent" id="BPMNPlane_timedBoundaryEvent">
      <bpmndi:BPMNShape bpmnElement="timedBoundaryStart" id="BPMNShape_timedBoundaryStart">
        <omgdc:Bounds height="30.0" width="30.0" x="120.0" y="137.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="deptApprove" id="BPMNShape_deptApprove">
        <omgdc:Bounds height="80.0" width="100.0" x="233.66666666666669" y="112.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="hrApprove" id="BPMNShape_hrApprove">
        <omgdc:Bounds height="80.0" width="100.0" x="420.0" y="112.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="manageApprove" id="BPMNShape_manageApprove">
        <omgdc:Bounds height="80.0" width="100.0" x="345.0" y="255.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="timedBoundaryId" id="BPMNShape_timedBoundaryId">
        <omgdc:Bounds height="31.0" width="31.0" x="256.132723174949" y="176.6130996333311"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-D9E71CF8-CABF-445A-957C-024AEB9D7F51" id="BPMNShape_sid-D9E71CF8-CABF-445A-957C-024AEB9D7F51">
        <omgdc:Bounds height="28.0" width="28.0" x="600.0" y="138.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-F6F7372A-2378-41E6-8712-4A602A923B22" id="BPMNEdge_sid-F6F7372A-2378-41E6-8712-4A602A923B22">
        <omgdi:waypoint x="150.0" y="152.0"></omgdi:waypoint>
        <omgdi:waypoint x="233.66666666666669" y="152.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-4DB7A535-3C11-4931-87DA-4DB6037D0A90" id="BPMNEdge_sid-4DB7A535-3C11-4931-87DA-4DB6037D0A90">
        <omgdi:waypoint x="272.132723174949" y="208.6130996333311"></omgdi:waypoint>
        <omgdi:waypoint x="272.132723174949" y="295.0"></omgdi:waypoint>
        <omgdi:waypoint x="345.0" y="295.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-F621A86A-13E3-4D59-82CB-4670F6331C10" id="BPMNEdge_sid-F621A86A-13E3-4D59-82CB-4670F6331C10">
        <omgdi:waypoint x="520.0" y="152.0"></omgdi:waypoint>
        <omgdi:waypoint x="600.0" y="152.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-7601B3F7-54E1-46F0-A829-3594101FC3DD" id="BPMNEdge_sid-7601B3F7-54E1-46F0-A829-3594101FC3DD">
        <omgdi:waypoint x="333.6666666666667" y="152.0"></omgdi:waypoint>
        <omgdi:waypoint x="420.0" y="152.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

### 2.2 启动流程

流程部署啥的步骤不介绍了，相信大家都已经很熟悉了。

![image-20200830221120817](/images/activiti6-19/image-20200830221120817.png)

### 2.3 验证

![image-20200830222234809](/images/activiti6-19/image-20200830222234809.png)

我们使用`leaderuser`登录，可以看到有一条待办

![image-20200830221338449](/images/activiti6-19/image-20200830221338449.png)

使用`manager`登录，没有待办

![image-20200830221451578](/images/activiti6-19/image-20200830221451578.png)

**五分钟后** 再次查看2人的待办

发现部门`manager`多了一条待办，而原先`leaderuser`的待办不见了（因为我们设置了cancelActivity=true）

![image-20200830224953816](/images/activiti6-19/image-20200830224953816.png)

此时我们查看流程图，可以看到已经到达了manager审批节点：

![image-20200830225346090](/images/activiti6-19/image-20200830225346090.png)

## 3、信号边界事件

依附在活动的边界上的信号捕获中间（事件），简称**信号边界事件**，捕获与其信号定义具有相同信号名的信号。

**请注意：**

与其他事件例如错误边界事件**不同**的是，信号边界事件不只是捕获其所依附范围抛出的信号。信号边界事件默认为全局范围（广播）的，意味着信号可以从任何地方抛出，甚至是不同的流程实例。

而且信号在被捕获后不会被消耗。如果有两个激活的信号边界事件，捕获相同的信号事件，则两个边界事件都会被触发，哪怕它们不在同一个流程实例里。（大白话就是一个抛出的信号，可以触发多个信号边界事件）

### 3.1  流程定义

我们依然自己画一个简单流程图来验证：

流程发起后，经过一个并行网关，部门经理审批后，到达**中间信号抛出事件**抛出一个信号，我们观察HR审批节点的边界信号事件，是否可以捕获这个信号，并中断原流程走向。

![image-20200831153320239](/images/activiti6-19/image-20200831153320239.png)

bpmn流程定义如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <signal id="signal001" name="signal001" activiti:scope="processInstance"></signal>
  <process id="signalBoundaryEvent" name="信号边界事件" isExecutable="true">
    <startEvent id="signalBoundaryStart" name="发起" activiti:initiator="applyUserId"></startEvent>
    <parallelGateway id="sid-1A318B96-1744-4C49-915D-DB26B68FD210"></parallelGateway>
    <sequenceFlow id="sid-D2E10E20-0BDF-4088-BC50-8904FF757FE8" sourceRef="signalBoundaryStart" targetRef="sid-1A318B96-1744-4C49-915D-DB26B68FD210"></sequenceFlow>
    <intermediateThrowEvent id="sid-367A1EB9-6546-4129-A352-F1327AC608A0" name="抛出信号">
      <signalEventDefinition signalRef="signal001"></signalEventDefinition>
    </intermediateThrowEvent>
    <endEvent id="sid-D4AC7877-A838-4DC8-9C03-8797F4C0D6A4"></endEvent>
    <userTask id="sid-B1358EF4-6DDB-4900-A555-6913D5ECD0EE" name="HR审批" activiti:candidateGroups="hr"></userTask>
    <boundaryEvent id="sid-D811D7A7-B27B-4C8D-AEBC-0EB11C1B6B23" attachedToRef="sid-B1358EF4-6DDB-4900-A555-6913D5ECD0EE" cancelActivity="true">
      <signalEventDefinition signalRef="signal001"></signalEventDefinition>
    </boundaryEvent>
    <sequenceFlow id="sid-6928F9A4-DE17-446E-8A8C-ABBC5A8905BD" sourceRef="sid-B1358EF4-6DDB-4900-A555-6913D5ECD0EE" targetRef="sid-D4AC7877-A838-4DC8-9C03-8797F4C0D6A4"></sequenceFlow>
    <userTask id="sid-7A35814D-2C06-4661-B740-6603C99734F6" name="manage审批" activiti:candidateGroups="generalManager"></userTask>
    <sequenceFlow id="sid-EB37508D-7DEE-43F5-99A1-A9E5EE8D0984" sourceRef="sid-D811D7A7-B27B-4C8D-AEBC-0EB11C1B6B23" targetRef="sid-7A35814D-2C06-4661-B740-6603C99734F6"></sequenceFlow>
    <userTask id="sid-500FA115-64AB-4DA8-862F-C97D9C06726B" name="部门经理审批" activiti:candidateGroups="deptLeader"></userTask>
    <sequenceFlow id="sid-0B6766D2-AE7E-4B8F-9D6B-C01C3135F51E" sourceRef="sid-1A318B96-1744-4C49-915D-DB26B68FD210" targetRef="sid-500FA115-64AB-4DA8-862F-C97D9C06726B"></sequenceFlow>
    <sequenceFlow id="sid-8139CF50-A8E2-4137-B05D-E1E994E028FF" sourceRef="sid-500FA115-64AB-4DA8-862F-C97D9C06726B" targetRef="sid-367A1EB9-6546-4129-A352-F1327AC608A0"></sequenceFlow>
    <sequenceFlow id="sid-D0901176-F889-4737-944E-38D3747C2BAA" sourceRef="sid-1A318B96-1744-4C49-915D-DB26B68FD210" targetRef="sid-B1358EF4-6DDB-4900-A555-6913D5ECD0EE"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_signalBoundaryEvent">
    <bpmndi:BPMNPlane bpmnElement="signalBoundaryEvent" id="BPMNPlane_signalBoundaryEvent">
      <bpmndi:BPMNShape bpmnElement="signalBoundaryStart" id="BPMNShape_signalBoundaryStart">
        <omgdc:Bounds height="30.0" width="30.0" x="120.0" y="225.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-1A318B96-1744-4C49-915D-DB26B68FD210" id="BPMNShape_sid-1A318B96-1744-4C49-915D-DB26B68FD210">
        <omgdc:Bounds height="40.0" width="40.0" x="255.0" y="220.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-367A1EB9-6546-4129-A352-F1327AC608A0" id="BPMNShape_sid-367A1EB9-6546-4129-A352-F1327AC608A0">
        <omgdc:Bounds height="30.0" width="30.0" x="540.0" y="145.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-D4AC7877-A838-4DC8-9C03-8797F4C0D6A4" id="BPMNShape_sid-D4AC7877-A838-4DC8-9C03-8797F4C0D6A4">
        <omgdc:Bounds height="28.0" width="28.0" x="541.0" y="326.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-B1358EF4-6DDB-4900-A555-6913D5ECD0EE" id="BPMNShape_sid-B1358EF4-6DDB-4900-A555-6913D5ECD0EE">
        <omgdc:Bounds height="80.0" width="100.0" x="364.0523247518181" y="300.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-D811D7A7-B27B-4C8D-AEBC-0EB11C1B6B23" id="BPMNShape_sid-D811D7A7-B27B-4C8D-AEBC-0EB11C1B6B23">
        <omgdc:Bounds height="30.0" width="30.0" x="401.102262368762" y="365.99875233887786"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-7A35814D-2C06-4661-B740-6603C99734F6" id="BPMNShape_sid-7A35814D-2C06-4661-B740-6603C99734F6">
        <omgdc:Bounds height="80.0" width="100.0" x="364.0523247518181" y="465.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-500FA115-64AB-4DA8-862F-C97D9C06726B" id="BPMNShape_sid-500FA115-64AB-4DA8-862F-C97D9C06726B">
        <omgdc:Bounds height="80.0" width="100.0" x="364.0523247518181" y="120.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-EB37508D-7DEE-43F5-99A1-A9E5EE8D0984" id="BPMNEdge_sid-EB37508D-7DEE-43F5-99A1-A9E5EE8D0984">
        <omgdi:waypoint x="415.8543224170552" y="395.99670305823776"></omgdi:waypoint>
        <omgdi:waypoint x="414.7135882973418" y="465.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-0B6766D2-AE7E-4B8F-9D6B-C01C3135F51E" id="BPMNEdge_sid-0B6766D2-AE7E-4B8F-9D6B-C01C3135F51E">
        <omgdi:waypoint x="275.5" y="220.5"></omgdi:waypoint>
        <omgdi:waypoint x="275.5" y="160.0"></omgdi:waypoint>
        <omgdi:waypoint x="364.0523247518181" y="160.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-8139CF50-A8E2-4137-B05D-E1E994E028FF" id="BPMNEdge_sid-8139CF50-A8E2-4137-B05D-E1E994E028FF">
        <omgdi:waypoint x="464.0523247518181" y="160.0"></omgdi:waypoint>
        <omgdi:waypoint x="540.0" y="160.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-D2E10E20-0BDF-4088-BC50-8904FF757FE8" id="BPMNEdge_sid-D2E10E20-0BDF-4088-BC50-8904FF757FE8">
        <omgdi:waypoint x="150.0" y="240.0"></omgdi:waypoint>
        <omgdi:waypoint x="255.0" y="240.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-D0901176-F889-4737-944E-38D3747C2BAA" id="BPMNEdge_sid-D0901176-F889-4737-944E-38D3747C2BAA">
        <omgdi:waypoint x="275.5" y="259.5"></omgdi:waypoint>
        <omgdi:waypoint x="275.5" y="340.0"></omgdi:waypoint>
        <omgdi:waypoint x="364.0523247518181" y="340.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-6928F9A4-DE17-446E-8A8C-ABBC5A8905BD" id="BPMNEdge_sid-6928F9A4-DE17-446E-8A8C-ABBC5A8905BD">
        <omgdi:waypoint x="464.0523247518181" y="340.0"></omgdi:waypoint>
        <omgdi:waypoint x="541.0" y="340.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

### 3.2 发起流程

![image-20200831154206232](/images/activiti6-19/image-20200831154206232.png)

![image-20200831153302662](/images/activiti6-19/image-20200831153302662.png)

### 3.3 验证

我们使用`leaderuser`签收、完成任务，之后流程流转到**中间信号抛出事件**并抛出信号。此时再次查看流程图即可发现，信号边界事件捕获了信号，并流转到了manage审批节点。符合预期~

![image-20200831154457619](/images/activiti6-19/image-20200831154457619.png)

### 3.4 API 触发型号边界事件

另外，本例中我们是通过信号抛出事件来抛出信号，除此外，我们还可以使用API的方式抛出信号：

```java 
RuntimeService.signalEventReceived(String signalName);
RuntimeService.signalEventReceived(String signalName, String executionId);
```

这2个方法应该表熟悉了，在信号启动事件中我们已经见到过了。我们本地实际操作以下：

依然是上面的流程，我们发起一个新的流程实例，但是不使用**中间信号抛出事件**来触发信号边界事件。

![image-20200901151018125](/images/activiti6-19/image-20200901151018125.png)

![image-20200901151102270](/images/activiti6-19/image-20200901151102270.png)

接下来我们使用API触发事件：

```java
RuntimeService.signalEventReceived(String signalName);
RuntimeService.signalEventReceived(String signalName, String executionId);
```

第一种，会触发全局范围内的信号事件，如果想要精准控制，就需要用到第二种。我们来获取`executionId`参数。

#### 3.4.1 获取某一信号事件的执行（Execution）

```java
/**
 * 获取信号事件的执行列表
*/
@PostMapping(value = "/getSignalEventSubscription/{processInstanceId}/{signalId}")
@ApiOperation(value = "获取某一信号事件的所有执行",notes = "获取某一信号事件的所有执行")
public CommonResponse signalEventSubscriptionName(@RequestParam(value = "pageNum", required = true,defaultValue = "1")@ApiParam(value = "页码" ,required = true)int pageNum,
                                                  @RequestParam(value = "pageSize", required = true,defaultValue = "10")@ApiParam(value = "条数" ,required = true)int pageSize,
                                                  @PathVariable(value ="processInstanceId", required = true) @ApiParam(value = "流程实例",required = false)String processInstanceId,
                                                  @PathVariable(value ="signalId", required = true) @ApiParam(value = "信号定义的编号",required = true)String signalId)  {
    Page page = processService.signalEventSubscriptionName(pageNum, pageSize, signalId, processInstanceId);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(page);
}
```

```java
/**
     * 获取信号事件的执行列表
     * @param pageNum
     * @param pageSize
     * @param signalId
     * @param processInstanceId
     * @return
     * @throws CommonException
     */
@Override
public Page signalEventSubscriptionName(int pageNum, int pageSize, String signalId,String processInstanceId)  {
    Page page = new Page(pageNum,pageSize);

    UserQueryImpl user = new UserQueryImpl();
    user = (UserQueryImpl)identityService.createUserQuery().userId(GlobalConfig.getOperator());

    List<AcExecutionEntityImpl> Acexecutions = new ArrayList<AcExecutionEntityImpl>();

    ExecutionQuery executionQuery = runtimeService.createExecutionQuery();
    if (!processInstanceId.isEmpty()){
        executionQuery = executionQuery.processInstanceId(processInstanceId);
    }

    List<Execution> executions = executionQuery
        .signalEventSubscriptionName(signalId)
        .listPage(page.getFirstResult(),page.getMaxResults());

    for (int i = 0; i <executions.size(); i++) {
        ExecutionEntityImpl execution = (ExecutionEntityImpl) executions.get(i);
        AcExecutionEntityImpl acExecutionEntity = new AcExecutionEntityImpl();

        acExecutionEntity.setId(execution.getId());
        acExecutionEntity.setActivitiId(execution.getActivityId());
        acExecutionEntity.setActivitiName(execution.getActivityName());
        acExecutionEntity.setParentId(execution.getParentId());
        acExecutionEntity.setProcessDefinitionId(execution.getProcessDefinitionId());
        acExecutionEntity.setProcessDefinitionKey(execution.getProcessDefinitionKey());
        acExecutionEntity.setRootProcessInstanceId(execution.getRootProcessInstanceId());
        acExecutionEntity.setProcessInstanceId(execution.getProcessInstanceId());
        Acexecutions.add(acExecutionEntity);
    }

    //获取总页数
    int total = (int) executionQuery.count();
    page.setTotal(total);
    page.setList(Acexecutions);
    return page;
}
```

以上代码可以让我们获取某个流程实例中的待触发的信号事件。我们来验证一下，

首先我们需要获取一下processInstanceId参数：140001（前面发起流程时已经返回了，实际编码中应该通过下面的接口获取）

![image-20200901151727374](/images/activiti6-19/image-20200901151727374.png)

然后我们获取一下信号事件的执行列表：

![image-20200901152045201](/images/activiti6-19/image-20200901152045201.png)

最后，调用API触发，此处我们改造一下下面的这个接口，让它同时支持**信号启动事件**和**信号边界事件**：

![image-20200901154759304](/images/activiti6-19/image-20200901154759304.png)

修改后的代码如下：

```java
/**
     * 信号触发
     */
@PostMapping(value = "/signalStartEventInstance/{signalName}")
@ApiOperation(value = "信号触发",notes = "可以用来发起启动节点是信号启动类型的流程，也可以用来触发信号边界事件，表单元素key需要以fp_开头")
public CommonResponse signalStartEventInstance(@PathVariable(value = "signalName",required = true) @ApiParam(value = "信号定义的编号",required = true)String signalName,
                                               @RequestParam(value = "executionId",required = false) @ApiParam(value = "执行ID",required = false)String executionId,
                                               HttpServletRequest request) {
    processService.signalStartEventInstance(signalName,executionId,request);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("信号触发成功");
}
```

```java
 /**
     * 信号触发
     * @param signalName
     * @param executionId
     * @param request
     * @throws CommonException
     */
@Override
public void signalStartEventInstance(String signalName,  String executionId ,HttpServletRequest request)  {
    Map<String, Object> formProperties = new HashMap<String, Object>();
    // 从request中读取参数然后转换
    Map<String, String[]> parameterMap = request.getParameterMap();
    if (parameterMap.size()>0){
        Set<Map.Entry<String, String[]>> entrySet = parameterMap.entrySet();
        for (Map.Entry<String, String[]> entry : entrySet) {
            String key = entry.getKey();
            // fp_的意思是form paremeter
            if (StringUtils.defaultString(key).startsWith("fp_")) {
                formProperties.put(key.split("_")[1], entry.getValue()[0]);
            }
        }
    }

    log.debug("start form parameters: {}", formProperties);

    UserQueryImpl user = new UserQueryImpl();
    user = (UserQueryImpl)identityService.createUserQuery().userId(GlobalConfig.getOperator());

    if (!executionId.isEmpty() && formProperties.size() > 0){
        runtimeService.signalEventReceived(signalName,executionId,formProperties);
    }else if(!executionId.isEmpty()){
        runtimeService.signalEventReceived(signalName,executionId);
    }else{
        runtimeService.signalEventReceived(signalName);
    }
}
```



因为不涉及表单字段，无需使用postman来测试，直接使用swagger来执行触发操作![image-20200901162207039](/images/activiti6-19/image-20200901162207039.png)

再次查看流程图：

![image-20200901163035231](/images/activiti6-19/image-20200901163035231.png)

#### 3.4.2 Scope作用域

上图**发现信号边界事件并没有生效，为啥呢？？**这里就涉及到了我们之前介绍过的信号的Scope作用域（信号开始事件时介绍过），因为我们在流程设计时`signal001`的`Scope`为`processInstance`，表示这个信号只能在当前流程中起作用，及只可以是本流程抛出的信号才能被捕获。

我们修改流程定义中的`Scope`，设置为global：

![image-20200901163927460](/images/activiti6-19/image-20200901163927460.png)

再次部署新的流程，并重复上面的操作：

![image-20200901164112738](/images/activiti6-19/image-20200901164112738.png)

![image-20200901164129974](/images/activiti6-19/image-20200901164129974.png)

可以看到，信号边界事件成功捕获了API抛出的信号。

