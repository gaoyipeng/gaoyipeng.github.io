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

#### 3.4.2 获取信号事件的执行列表

![image-20200901152045201](/images/activiti6-19/image-20200901152045201.png)

#### 3.4.3 旧接口改造

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

#### 3.4.4 API触发

因为不涉及表单字段，无需使用postman来测试，直接使用swagger来执行触发操作![image-20200901162207039](/images/activiti6-19/image-20200901162207039.png)

再次查看流程图：

![image-20200901163035231](/images/activiti6-19/image-20200901163035231.png)

#### 3.4.5 Scope作用域

上图**发现信号边界事件并没有生效，为啥呢？？**这里就涉及到了我们之前介绍过的信号的Scope作用域（信号开始事件时介绍过），因为我们在流程设计时`signal001`的`Scope`为`processInstance`，表示这个信号只能在当前流程中起作用，及只可以是本流程抛出的信号才能被捕获。

我们修改流程定义中的`Scope`，设置为global：

![image-20200901163927460](/images/activiti6-19/image-20200901163927460.png)

再次部署新的流程，并重复上面的操作：

![image-20200901164112738](/images/activiti6-19/image-20200901164112738.png)

![image-20200901164129974](/images/activiti6-19/image-20200901164129974.png)

可以看到，信号边界事件成功捕获了API抛出的信号。

## 4、消息边界事件

**消息边界事件**，捕获与其消息定义具有相同消息名的消息。

消息边界事件和信号边界事件很相似，**不同的是**信号边界事件既可以通过**中间信号抛出事件**来触发，也可以使用API方式触发。而消息边界事件只能通过API方式触发。

### 4.1 流程定义

我们依然画一个简单的流程图来验证一下：

![image-20200902135737156](/images/activiti6-19/image-20200902135737156.png)

![image-20200902142039098](/images/activiti6-19/image-20200902142039098.png)

bpmn流程定义：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <message id="message01" name="message01"></message>
  <process id="messageBoundaryEvent" name="消息边界事件" isExecutable="true">
    <startEvent id="sid-170C9C8A-F77E-42B9-874D-55E335035D07"></startEvent>
    <parallelGateway id="sid-218D406B-55E9-464B-B076-702AE7AF3B80"></parallelGateway>
    <sequenceFlow id="sid-3EED251F-5504-4D57-AB7D-77EF2731ADE6" sourceRef="sid-170C9C8A-F77E-42B9-874D-55E335035D07" targetRef="sid-218D406B-55E9-464B-B076-702AE7AF3B80"></sequenceFlow>
    <userTask id="sid-5F455106-F1CE-457A-9FB5-32D7A361CD98" name="部门经理审批" activiti:candidateGroups="deptLeader"></userTask>
    <sequenceFlow id="sid-0AB4B167-C0E9-4F57-A18D-F041C1ACA4F7" sourceRef="sid-218D406B-55E9-464B-B076-702AE7AF3B80" targetRef="sid-5F455106-F1CE-457A-9FB5-32D7A361CD98"></sequenceFlow>
    <userTask id="sid-9C72719B-59D2-4F1A-934E-885828AE42AC" name="HR审批" activiti:candidateGroups="hr"></userTask>
    <sequenceFlow id="sid-B888CFA6-66A5-4E6D-B4D5-0F6426330FDB" sourceRef="sid-218D406B-55E9-464B-B076-702AE7AF3B80" targetRef="sid-9C72719B-59D2-4F1A-934E-885828AE42AC"></sequenceFlow>
    <userTask id="sid-18CB148C-D621-409F-A7CF-064F4A292193" name="manage审批" activiti:candidateGroups="generalManager"></userTask>
    <endEvent id="sid-0B1FBE89-725A-4952-810E-F1193F60BB81"></endEvent>
    <sequenceFlow id="sid-AFC2FD4C-58A2-4FF3-84E9-452DB0373E72" sourceRef="sid-9C72719B-59D2-4F1A-934E-885828AE42AC" targetRef="sid-0B1FBE89-725A-4952-810E-F1193F60BB81"></sequenceFlow>
    <sequenceFlow id="sid-9289A120-5DE9-423C-8698-9E4F41B8396B" sourceRef="sid-18CB148C-D621-409F-A7CF-064F4A292193" targetRef="sid-0B1FBE89-725A-4952-810E-F1193F60BB81"></sequenceFlow>
    <sequenceFlow id="sid-DC0FEF87-A08E-4B3A-AA1A-1D6341A563EE" sourceRef="sid-BE932A79-1947-4FF2-94FB-F3A04F7D7059" targetRef="sid-18CB148C-D621-409F-A7CF-064F4A292193"></sequenceFlow>
    <boundaryEvent id="sid-BE932A79-1947-4FF2-94FB-F3A04F7D7059" attachedToRef="sid-9C72719B-59D2-4F1A-934E-885828AE42AC" cancelActivity="false">
      <messageEventDefinition messageRef="message01"></messageEventDefinition>
    </boundaryEvent>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_messageBoundaryEvent">
    <bpmndi:BPMNPlane bpmnElement="messageBoundaryEvent" id="BPMNPlane_messageBoundaryEvent">
      <bpmndi:BPMNShape bpmnElement="sid-170C9C8A-F77E-42B9-874D-55E335035D07" id="BPMNShape_sid-170C9C8A-F77E-42B9-874D-55E335035D07">
        <omgdc:Bounds height="30.0" width="30.0" x="125.89999999999998" y="215.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-218D406B-55E9-464B-B076-702AE7AF3B80" id="BPMNShape_sid-218D406B-55E9-464B-B076-702AE7AF3B80">
        <omgdc:Bounds height="40.0" width="40.0" x="210.0" y="210.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-5F455106-F1CE-457A-9FB5-32D7A361CD98" id="BPMNShape_sid-5F455106-F1CE-457A-9FB5-32D7A361CD98">
        <omgdc:Bounds height="80.0" width="100.0" x="345.0" y="105.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-9C72719B-59D2-4F1A-934E-885828AE42AC" id="BPMNShape_sid-9C72719B-59D2-4F1A-934E-885828AE42AC">
        <omgdc:Bounds height="80.0" width="100.0" x="345.0" y="285.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-18CB148C-D621-409F-A7CF-064F4A292193" id="BPMNShape_sid-18CB148C-D621-409F-A7CF-064F4A292193">
        <omgdc:Bounds height="80.0" width="100.0" x="340.33218883279585" y="435.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-0B1FBE89-725A-4952-810E-F1193F60BB81" id="BPMNShape_sid-0B1FBE89-725A-4952-810E-F1193F60BB81">
        <omgdc:Bounds height="28.0" width="28.0" x="570.0" y="311.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-BE932A79-1947-4FF2-94FB-F3A04F7D7059" id="BPMNShape_sid-BE932A79-1947-4FF2-94FB-F3A04F7D7059">
        <omgdc:Bounds height="30.0" width="30.0" x="375.33218883279585" y="350.7654312023694"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-AFC2FD4C-58A2-4FF3-84E9-452DB0373E72" id="BPMNEdge_sid-AFC2FD4C-58A2-4FF3-84E9-452DB0373E72">
        <omgdi:waypoint x="445.0" y="325.0"></omgdi:waypoint>
        <omgdi:waypoint x="570.0" y="325.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-DC0FEF87-A08E-4B3A-AA1A-1D6341A563EE" id="BPMNEdge_sid-DC0FEF87-A08E-4B3A-AA1A-1D6341A563EE">
        <omgdi:waypoint x="390.33218883279585" y="380.7654312023694"></omgdi:waypoint>
        <omgdi:waypoint x="390.33218883279585" y="435.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-3EED251F-5504-4D57-AB7D-77EF2731ADE6" id="BPMNEdge_sid-3EED251F-5504-4D57-AB7D-77EF2731ADE6">
        <omgdi:waypoint x="155.89976645256073" y="230.08370405386475"></omgdi:waypoint>
        <omgdi:waypoint x="210.38776655443323" y="230.38776655443323"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-0AB4B167-C0E9-4F57-A18D-F041C1ACA4F7" id="BPMNEdge_sid-0AB4B167-C0E9-4F57-A18D-F041C1ACA4F7">
        <omgdi:waypoint x="239.28688524590163" y="219.28688524590163"></omgdi:waypoint>
        <omgdi:waypoint x="297.5" y="145.0"></omgdi:waypoint>
        <omgdi:waypoint x="345.0" y="145.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-B888CFA6-66A5-4E6D-B4D5-0F6426330FDB" id="BPMNEdge_sid-B888CFA6-66A5-4E6D-B4D5-0F6426330FDB">
        <omgdi:waypoint x="237.8409090909091" y="242.1590909090909"></omgdi:waypoint>
        <omgdi:waypoint x="290.0" y="325.0"></omgdi:waypoint>
        <omgdi:waypoint x="345.0" y="325.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-9289A120-5DE9-423C-8698-9E4F41B8396B" id="BPMNEdge_sid-9289A120-5DE9-423C-8698-9E4F41B8396B">
        <omgdi:waypoint x="440.33218883279585" y="475.0"></omgdi:waypoint>
        <omgdi:waypoint x="584.0" y="475.0"></omgdi:waypoint>
        <omgdi:waypoint x="584.0" y="339.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

### 4.2 发起流程

![image-20200902140706434](/images/activiti6-19/image-20200902140706434.png)

![image-20200902140746066](/images/activiti6-19/image-20200902140746066.png)

### 4.3 API触发消息边界事件

Activiti提供了以下2个API 来抛出消息：

```java
void messageEventReceived(String messageName, String executionId);
void messageEventReceived(String messageName, String executionId, HashMap<String, Object> processVariables);
```

我们先来获取`executionId`参数。

#### 4.3.1 获取某一消息事件的执行（Execution）

以下代码可以让我们获取某个流程实例中的待触发的消息事件。

```java
/**
 * 获取消息事件的执行列表
*/
@PostMapping(value = "/getMessageEventSubscription/{processInstanceId}/{messageName}")
@ApiOperation(value = "获取某一消息事件的所有执行",notes = "获取某一消息事件的所有执行")
public CommonResponse messageEventSubscriptionName(@RequestParam(value = "pageNum", required = true,defaultValue = "1")@ApiParam(value = "页码" ,required = true)int pageNum,
                                                   @RequestParam(value = "pageSize", required = true,defaultValue = "10")@ApiParam(value = "条数" ,required = true)int pageSize,
                                                   @PathVariable(value ="processInstanceId", required = false) @ApiParam(value = "流程实例",required = false)String processInstanceId,
                                                   @PathVariable(value ="messageName", required = true) @ApiParam(value = "消息定义的名称",required = true)String messageName)  {
    Page page = processService.messageEventSubscriptionName(pageNum, pageSize, messageName, processInstanceId);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(page);
}
```

```java
/**
 * 获取消息事件的执行列表
 * @param pageNum
 * @param pageSize
 * @param messageName
 * @param processInstanceId
 * @return
*/
@Override
public Page messageEventSubscriptionName(int pageNum, int pageSize, String messageName, String processInstanceId) {
    Page page = new Page(pageNum,pageSize);

    UserQueryImpl user = new UserQueryImpl();
    user = (UserQueryImpl)identityService.createUserQuery().userId(GlobalConfig.getOperator());

    List<AcExecutionEntityImpl> Acexecutions = new ArrayList<AcExecutionEntityImpl>();

    ExecutionQuery executionQuery = runtimeService.createExecutionQuery();
    if (!processInstanceId.isEmpty()){
        executionQuery = executionQuery.processInstanceId(processInstanceId);
    }

    List<Execution> executions = executionQuery
        .messageEventSubscriptionName(messageName)
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

首先我们需要获取一下processInstanceId参数：147501（前面发起流程时已经返回了，实际编码中应该通过下面的接口获取）

![image-20200902143219786](/images/activiti6-19/image-20200902143219786.png)

#### 4.3.2 获取消息事件的执行列表

![image-20200902143405319](/images/activiti6-19/image-20200902143405319.png)

#### 4.3.3 API触发

消息边界的触发与消息启动事件的API是不同的，我们单独写一个接口：

```java
/**
     * 消息触发
     */
@PostMapping(value = "/messageEventReceived/{messageName}")
@ApiOperation(value = "消息触发",notes = "消息触发，表单元素key需要以fp_开头")
public CommonResponse messageEventReceived(@PathVariable(value = "messageName",required = true) @ApiParam(value = "消息定义的名称",required = true)String messageName,
                                           @RequestParam(value = "executionId",required = false) @ApiParam(value = "执行ID",required = false)String executionId,
                                           HttpServletRequest request) throws CommonException {
    processService.messageEventReceived(messageName,executionId,request);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("消息触发成功");
}
```

```java
 /**
     * 消息触发
     * @param messageName
     * @param executionId
     * @param request
     */
@Override
public void messageEventReceived(String messageName, String executionId, HttpServletRequest request) {
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
        runtimeService.messageEventReceived(messageName,executionId,formProperties);
    }else if(!executionId.isEmpty()){
        runtimeService.messageEventReceived(messageName,executionId);
    }
}
```

此时我们已经获取了所需的messageName和executionId：

![image-20200902144507090](/images/activiti6-19/image-20200902144507090.png)

此时再看流程图，可以看到消息已经被消息边界事件捕获了：

![image-20200902144555281](/images/activiti6-19/image-20200902144555281.png)

#### 4.3.4 cancelActivity参数说明

本流程还有一个需要注意的地方，这个消息边界事件的外面2个圆圈是虚线（cancelActivity="false"），表示触发后原流程流转方向不会中断，例如本流程中**HR审批节点**依然可以审批，并结束流程。

若cancelActivity="true",则原流程环节就中断了，只能通过`manage审批`节点来继续此流程的审批。

![image-20200902144805834](/images/activiti6-19/image-20200902144805834.png)

我们使用hr审批流程来验证一下：

![image-20200902145509534](/images/activiti6-19/image-20200902145509534.png)

再次查看流程图，可以看到流程已经结束了。

![image-20200902145541799](/images/activiti6-19/image-20200902145541799.png)

## 5、边界补偿事件（边界修正事件）

**补偿边界事件**，可以为活动附加补偿处理器。

补偿边界事件必须通过直接关联的方式，引用单个的补偿处理器。

补偿边界事件与其它边界事件的活动策略不同。其它边界事件，例如信号边界事件，当其依附的活动启动时激活；当离开该活动时，会被解除，并取消相应的事件订阅。而补偿边界事件不是这样。补偿边界事件在其依附的活动**成功完成**时激活，同时创建补偿事件的相应订阅。当补偿事件被触发，或者相应的流程实例结束时，才会移除订阅。请考虑下列因素：

- 当补偿被触发时，补偿边界事件关联的补偿处理器会被调用，次数与其依附的活动成功完成的次数相同。
- 如果补偿边界事件依附在具有多实例特性的活动上，则会为每一个实例创建补偿事件订阅。
- 如果补偿边界事件依附在位于循环内部的活动上，则每次该活动执行时，都会创建一个补偿事件订阅。
- 如果流程实例结束，则取消补偿事件的订阅。

**请注意：**嵌入式子流程不支持补偿边界事件。

### 5.1 触发条件

- 事务子流程中**取消结束事件**触发**边界补偿事件**。

- **补偿中间抛出事件**触发**边界补偿事件**。

  
  
  第一种触发条件我们在事务子流程中一起介绍，这里只介绍第2种。

### 5.2 补偿中间抛出事件触发边界补偿事件

此处我们画一个流程：

![image-20200904111109326](/images/activiti6-19/image-20200904111109326.png)

说明：这是简单的付款流程，流程启动后按照：A付款——B收钱——多实例——补偿中间抛出事件——结束的顺序来执行。

另外3个点为补偿操作，通过补偿边界事件与主流程环节相连。假设流程出现错误，需要回滚，补偿中间抛出事件会触发第二行的3个补偿操作。

`多实例`节点我们设置了3个并行操作。

![image-20200904101937066](/images/activiti6-19/image-20200904101937066.png)

#### 5.2.1 流程定义

下面是完整的bpmn定义：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/test">
  <process id="boundaryCompensationEvent" name="边界补偿事件" isExecutable="true">
    <startEvent id="startevent1" name="Start"></startEvent>
    <serviceTask id="servicetask1" name="A付款" activiti:class="com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent.APaymentService"></serviceTask>
    <serviceTask id="servicetask2" name="B收钱" activiti:class="com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent.BReceivePaymentService"></serviceTask>
    <intermediateThrowEvent id="compensationintermediatethrowevent1" name="CompensationThrowingEvent">
		<compensateEventDefinition></compensateEventDefinition>
	</intermediateThrowEvent>
    <endEvent id="endevent1" name="End"></endEvent>
    <sequenceFlow id="flow1" sourceRef="startevent1" targetRef="servicetask1"></sequenceFlow>
    <sequenceFlow id="flow2" sourceRef="servicetask1" targetRef="servicetask2"></sequenceFlow>
    <sequenceFlow id="flow4" sourceRef="compensationintermediatethrowevent1" targetRef="endevent1"></sequenceFlow>
    <serviceTask id="servicetask3" name="补偿操作（A撤销付款操作）" activiti:class="com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent.AMakeUpService" isForCompensation="true"></serviceTask>
    <serviceTask id="servicetask4" name="补偿操作（B撤销收钱操作）" activiti:class="com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent.BMakeUpService" isForCompensation="true"></serviceTask>
    <serviceTask id="sid-05354E87-3CC4-4F1C-9C73-A2816D33D1CF" name="多实例补偿" activiti:class="com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent.MultiInstanceMakeUpService" isForCompensation="true"></serviceTask>
    <serviceTask id="sid-2310FA00-EFE0-4E98-AA31-E2AEF1DD1AD6" name="多实例" activiti:class="com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent.MultiInstanceService">
      <multiInstanceLoopCharacteristics isSequential="false">
        <loopCardinality>3</loopCardinality>
      </multiInstanceLoopCharacteristics>
    </serviceTask>
    <sequenceFlow id="sid-B628EA27-7AC3-4D84-BF02-088D1CC92C19" sourceRef="servicetask2" targetRef="sid-2310FA00-EFE0-4E98-AA31-E2AEF1DD1AD6"></sequenceFlow>
    <sequenceFlow id="sid-1948505E-385D-4AB5-8FD6-8ADC4507B559" sourceRef="sid-2310FA00-EFE0-4E98-AA31-E2AEF1DD1AD6" targetRef="compensationintermediatethrowevent1"></sequenceFlow>
    <boundaryEvent id="sid-B9CFEB1B-2A60-440C-AA13-6F74DD8E6A7F" attachedToRef="sid-2310FA00-EFE0-4E98-AA31-E2AEF1DD1AD6" cancelActivity="false">
      <compensateEventDefinition></compensateEventDefinition>
    </boundaryEvent>
    <boundaryEvent id="boundarycompensation2" attachedToRef="servicetask2" cancelActivity="true">
      <compensateEventDefinition></compensateEventDefinition>
    </boundaryEvent>
    <boundaryEvent id="boundarycompensation1" attachedToRef="servicetask1" cancelActivity="true">
      <compensateEventDefinition></compensateEventDefinition>
    </boundaryEvent>
    <association id="association1" sourceRef="boundarycompensation1" targetRef="servicetask3" associationDirection="None"></association>
    <association id="association2" sourceRef="boundarycompensation2" targetRef="servicetask4" associationDirection="None"></association>
    <association id="sid-65D72081-517C-4095-8D72-1785ED71CFEF" sourceRef="sid-B9CFEB1B-2A60-440C-AA13-6F74DD8E6A7F" targetRef="sid-05354E87-3CC4-4F1C-9C73-A2816D33D1CF" associationDirection="None"></association>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_boundaryCompensationEvent">
    <bpmndi:BPMNPlane bpmnElement="boundaryCompensationEvent" id="BPMNPlane_boundaryCompensationEvent">
      <bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
        <omgdc:Bounds height="30.0" width="30.0" x="70.0" y="155.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="servicetask1" id="BPMNShape_servicetask1">
        <omgdc:Bounds height="55.0" width="105.0" x="165.0" y="140.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="servicetask2" id="BPMNShape_servicetask2">
        <omgdc:Bounds height="55.0" width="105.0" x="340.0" y="140.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="compensationintermediatethrowevent1" id="BPMNShape_compensationintermediatethrowevent1">
        <omgdc:Bounds height="30.0" width="30.0" x="765.0" y="155.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
        <omgdc:Bounds height="28.0" width="28.0" x="900.0" y="156.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="servicetask3" id="BPMNShape_servicetask3">
        <omgdc:Bounds height="80.0" width="134.99999999999997" x="159.46846851919068" y="270.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="servicetask4" id="BPMNShape_servicetask4">
        <omgdc:Bounds height="71.0" width="135.0" x="327.05801075438586" y="277.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="association1" id="BPMNShape_association1">
        <omgdc:Bounds height="58.796701737861724" width="0.7432082881786926" x="226.22526023101196" y="211.20329826213828"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="association2" id="BPMNShape_association2">
        <omgdc:Bounds height="66.80624613905798" width="0.0" x="394.55801075438586" y="210.19375386094202"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-05354E87-3CC4-4F1C-9C73-A2816D33D1CF" id="BPMNShape_sid-05354E87-3CC4-4F1C-9C73-A2816D33D1CF">
        <omgdc:Bounds height="71.0" width="145.0" x="531.0698320860189" y="274.5"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-65D72081-517C-4095-8D72-1785ED71CFEF" id="BPMNShape_sid-65D72081-517C-4095-8D72-1785ED71CFEF">
        <omgdc:Bounds height="47.73368688736619" width="0.0" x="603.5698320860189" y="226.29731451995437"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-2310FA00-EFE0-4E98-AA31-E2AEF1DD1AD6" id="BPMNShape_sid-2310FA00-EFE0-4E98-AA31-E2AEF1DD1AD6">
        <omgdc:Bounds height="80.0" width="100.0" x="535.4" y="130.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-B9CFEB1B-2A60-440C-AA13-6F74DD8E6A7F" id="BPMNShape_sid-B9CFEB1B-2A60-440C-AA13-6F74DD8E6A7F">
        <omgdc:Bounds height="30.0" width="30.0" x="588.5698320860189" y="195.5326262252675"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="boundarycompensation2" id="BPMNShape_boundarycompensation2">
        <omgdc:Bounds height="30.0" width="30.0" x="379.55801075438586" y="180.12075937925076"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="boundarycompensation1" id="BPMNShape_boundarycompensation1">
        <omgdc:Bounds height="30.0" width="30.0" x="211.02916278342167" y="180.6896349716271"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="association1" id="BPMNEdge_association1">
        <omgdi:waypoint x="226.21875222451516" y="210.68843678523294"></omgdi:waypoint>
        <omgdi:waypoint x="226.96846851919065" y="270.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow1" id="BPMNEdge_flow1">
        <omgdi:waypoint x="99.99733072335283" y="169.7170314957858"></omgdi:waypoint>
        <omgdi:waypoint x="165.0" y="168.49056603773585"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="association2" id="BPMNEdge_association2">
        <omgdi:waypoint x="394.55801075438586" y="210.12075937925076"></omgdi:waypoint>
        <omgdi:waypoint x="394.55801075438586" y="277.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2">
        <omgdi:waypoint x="270.0" y="167.5"></omgdi:waypoint>
        <omgdi:waypoint x="340.0" y="167.5"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-65D72081-517C-4095-8D72-1785ED71CFEF" id="BPMNEdge_sid-65D72081-517C-4095-8D72-1785ED71CFEF">
        <omgdi:waypoint x="603.5698320860189" y="225.5326262252675"></omgdi:waypoint>
        <omgdi:waypoint x="603.5698320860189" y="274.5"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow4" id="BPMNEdge_flow4">
        <omgdi:waypoint x="795.0" y="170.0"></omgdi:waypoint>
        <omgdi:waypoint x="900.0" y="170.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-1948505E-385D-4AB5-8FD6-8ADC4507B559" id="BPMNEdge_sid-1948505E-385D-4AB5-8FD6-8ADC4507B559">
        <omgdi:waypoint x="635.4" y="170.0"></omgdi:waypoint>
        <omgdi:waypoint x="765.0" y="170.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-B628EA27-7AC3-4D84-BF02-088D1CC92C19" id="BPMNEdge_sid-B628EA27-7AC3-4D84-BF02-088D1CC92C19">
        <omgdi:waypoint x="445.0" y="168.18040435458786"></omgdi:waypoint>
        <omgdi:waypoint x="535.4" y="169.35199585277346"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

#### 5.2.2 手动修改流程定义

这里说明一下：直接使用流程设计器设计的流程图有些问题，下图中标出的4个地方，是后面手动加上去的，目前的流程设计器不支持这几个选项。（可能是这个版本的流程设计器的问题，后期再看看是否能修改）

![image-20200904093012762](/images/activiti6-19/image-20200904093012762.png)

第一处：

`<cancelEventDefinition></cancelEventDefinition>`表示这是一个**中间抛出补偿事件**，而目前流程设计器只支持无触发器的中间抛出事件。

![image-20200904095614642](/images/activiti6-19/image-20200904095614642.png)

```xml
<!--无触发器的中间抛出事件-->
<intermediateThrowEvent id="compensationintermediatethrowevent1" name="CompensationThrowingEvent">
</intermediateThrowEvent>

<!--中间抛出补偿事件 在上面的代码基础上包含了<compensateEventDefinition></compensateEventDefinition> -->
<intermediateThrowEvent id="compensationintermediatethrowevent1" name="CompensationThrowingEvent">
		<compensateEventDefinition></compensateEventDefinition>
</intermediateThrowEvent>
```

剩余三处：`isForCompensation="true"`表示这是一个补偿，加在3个补偿操作上

![image-20200904095802532](/images/activiti6-19/image-20200904095802532.png)

修改后重新部署即可

#### 5.2.3  节点服务任务

流程图中我们一共用到了6个Java Service任务 `<serviceTask>`。

![image-20200715155238237](/images/activiti6-19/image-20200715155238237.png)

此处我们使用的是第一种，这里我们贴出6个Java Service：

```java
package com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent;

import org.activiti.engine.delegate.DelegateExecution;
import org.activiti.engine.delegate.JavaDelegate;
import org.springframework.stereotype.Component;

@Component
public class APaymentService implements JavaDelegate {
    @Override
    public void execute(DelegateExecution delegateExecution) {
        System.out.println("A扣款~~~"+delegateExecution.getEventName());
    }
}
```

```java
package com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent;

import org.activiti.engine.delegate.DelegateExecution;
import org.activiti.engine.delegate.JavaDelegate;
import org.springframework.stereotype.Component;

@Component
public class AMakeUpService implements JavaDelegate {
    @Override
    public void execute(DelegateExecution delegateExecution) {
        System.out.println("A补偿~~~"+delegateExecution.getEventName());
    }
}
```

```java
package com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent;

import org.activiti.engine.delegate.DelegateExecution;
import org.activiti.engine.delegate.JavaDelegate;
import org.springframework.stereotype.Component;

@Component
public class BReceivePaymentService implements JavaDelegate {
    @Override
    public void execute(DelegateExecution delegateExecution) {
        System.out.println("B收款~~~"+delegateExecution.getEventName());
    }
}

```

```java
package com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent;

import org.activiti.engine.delegate.DelegateExecution;
import org.activiti.engine.delegate.JavaDelegate;
import org.springframework.stereotype.Component;

@Component
public class BMakeUpService implements JavaDelegate {
    @Override
    public void execute(DelegateExecution delegateExecution) {
        System.out.println("B补偿~~~"+delegateExecution.getEventName());
    }
}

```

```java
package com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent;

import org.activiti.engine.delegate.DelegateExecution;
import org.activiti.engine.delegate.JavaDelegate;
import org.springframework.stereotype.Component;

@Component
public class MultiInstanceService implements JavaDelegate {
    @Override
    public void execute(DelegateExecution delegateExecution) {
        System.out.println("多实例~~~"+delegateExecution.getEventName());
    }
}

```

```java
package com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent;

import org.activiti.engine.delegate.DelegateExecution;
import org.activiti.engine.delegate.JavaDelegate;
import org.springframework.stereotype.Component;

@Component
public class MultiInstanceMakeUpService implements JavaDelegate {
    @Override
    public void execute(DelegateExecution delegateExecution) {
        System.out.println("多实例补偿操作~~~"+delegateExecution.getEventName());
    }
}

```

#### 5.2.4 发起流程

![image-20200904100133263](/images/activiti6-19/image-20200904100133263.png)

我们流程图和打印结果：

![image-20200904101638227](/images/activiti6-19/image-20200904101638227.png)

```
A扣款~~~null
B收款~~~null
多实例~~~null
多实例~~~null
多实例~~~null
多实例补偿操作~~~null
多实例补偿操作~~~null
多实例补偿操作~~~null
B补偿~~~null
A补偿~~~null
```

可以看到补偿顺利执行了。

#### 5.2.5 执行顺序

我们通过查看打印结果即可得出流程的执行顺序：先执行后补偿，后执行先补偿。例如A先扣款B后收款，而在触发补偿后却是B先补偿A后补偿。

#### 5.2.6 多实例补偿规则

我们通过查看打印结果即可得出：

如果补偿边界事件依附在具有多实例特性的活动上，则会为每一个实例创建补偿事件订阅。

## 6、边界取消事件

**取消边界事件**是依附在事务子流程边界上的，使用**取消结束事件**在事务取消时触发。当取消边界事件触发时，首先会中断当前范围的所有活动执行。接下来，启动事务范围内所有有效的的补偿边界事件（compensation boundary event）。补偿会同步执行，也就是说在离开事务前，边界事件会等待补偿完成。当补偿完成时，使用取消边界事件的出口顺序流，离开事务子流程。

**取消边界事件**和**取消结束事件**一样，我们都在事务子流程中一起介绍。