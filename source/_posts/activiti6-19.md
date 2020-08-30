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

定时边界事件就是一个暂停等待警告的时钟。当流程执行到绑定了边界事件的环节， 会启动一个定时器。默认当定时器触发时（比如，一定时间之后），环节就会中断， 并沿着定时边界事件的外出连线继续执行。但是如果不想让原环节中断，可以将*cancelActivity*属性设置为**false**.

### 2.1 定义流程

我们此处定义一个简单流程来验证一下：

部门经理节点设置了一个定时边界事件，设置为5分钟后。并且取消活动打钩（cancelActivity=true）

![image-20200830215139310](/images/activiti6-19/image-20200830215139310.png)

![image-20200830215439567](/images/activiti6-19/image-20200830215439567.png)

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