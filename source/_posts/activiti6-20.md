---
title: Activiti（20） 中间事件
date: 2020-09-07 09:50:03
tags: Activiti
categories: Activiti
description: Activiti（20） 中间事件
typora-root-url: ..
password: kiki
---

 中间事件是指可以单独作为流程元素的事件，BPMN2.0规范中所指的中间事件也包括边界事件。

中间事件按照其特性可以分为两类：中间Catching(捕获)事件和中间Throwing(抛出)事件，当流程到达中间Catching事件时，它会一直在等待被触发，直接接收到的信息，才会被触发，而当流程到达中间Throwing事件时，该事件会自动被触发并抛出相应的结果或者信息。

## 1、定时器捕获中间事件

定时器中间事件是一个Catching事件，当执行到达捕获事件节点， 就会启动一个定时器。 当定时器触发（一段时间之后），流程就会沿着定时中间事件的外出节点继续执行。

此处我们设计一个简单流程来验证：

发起流程后到达定时器中间捕获事件，设置5分钟后流程继续执行。

![image-20200907113112480](/images/activiti6-20/image-20200907113112480.png)

bpmn流程定义：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <process id="timerCapturesIntermediateEvents" name="定时器捕获中间事件" isExecutable="true">
    <startEvent id="start"></startEvent>
    <intermediateCatchEvent id="sid-DBB0D1AA-199B-49BE-8FEC-9C032EDEF40C">
      <timerEventDefinition>
        <timeDuration>PT5M</timeDuration>
      </timerEventDefinition>
    </intermediateCatchEvent>
    <userTask id="sid-D04C33DE-965D-4A0B-948D-FABE89724AC0" name="部门经理审批" activiti:candidateGroups="deptLeader"></userTask>
    <sequenceFlow id="sid-FF8DB79B-EE0D-4E84-8D9E-9B31A7D23FA2" sourceRef="sid-DBB0D1AA-199B-49BE-8FEC-9C032EDEF40C" targetRef="sid-D04C33DE-965D-4A0B-948D-FABE89724AC0"></sequenceFlow>
    <endEvent id="sid-D9EEEB07-659D-4B9C-AE7A-E461497B9EEE"></endEvent>
    <sequenceFlow id="sid-A86BAC22-8FBC-4DB0-901B-241D77BCF0DB" sourceRef="sid-D04C33DE-965D-4A0B-948D-FABE89724AC0" targetRef="sid-D9EEEB07-659D-4B9C-AE7A-E461497B9EEE"></sequenceFlow>
    <sequenceFlow id="sid-85DAA48B-046E-4884-8C5F-8FC39FE6C515" sourceRef="start" targetRef="sid-DBB0D1AA-199B-49BE-8FEC-9C032EDEF40C"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_timerCapturesIntermediateEvents">
    <bpmndi:BPMNPlane bpmnElement="timerCapturesIntermediateEvents" id="BPMNPlane_timerCapturesIntermediateEvents">
      <bpmndi:BPMNShape bpmnElement="start" id="BPMNShape_start">
        <omgdc:Bounds height="30.0" width="30.0" x="210.0" y="127.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-DBB0D1AA-199B-49BE-8FEC-9C032EDEF40C" id="BPMNShape_sid-DBB0D1AA-199B-49BE-8FEC-9C032EDEF40C">
        <omgdc:Bounds height="31.0" width="31.0" x="315.0" y="126.5"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-D04C33DE-965D-4A0B-948D-FABE89724AC0" id="BPMNShape_sid-D04C33DE-965D-4A0B-948D-FABE89724AC0">
        <omgdc:Bounds height="80.0" width="100.0" x="421.0" y="102.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-D9EEEB07-659D-4B9C-AE7A-E461497B9EEE" id="BPMNShape_sid-D9EEEB07-659D-4B9C-AE7A-E461497B9EEE">
        <omgdc:Bounds height="28.0" width="28.0" x="566.0" y="128.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-FF8DB79B-EE0D-4E84-8D9E-9B31A7D23FA2" id="BPMNEdge_sid-FF8DB79B-EE0D-4E84-8D9E-9B31A7D23FA2">
        <omgdi:waypoint x="346.99989796015984" y="142.44285750728514"></omgdi:waypoint>
        <omgdi:waypoint x="421.0" y="142.17857142857144"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-A86BAC22-8FBC-4DB0-901B-241D77BCF0DB" id="BPMNEdge_sid-A86BAC22-8FBC-4DB0-901B-241D77BCF0DB">
        <omgdi:waypoint x="521.0" y="142.0"></omgdi:waypoint>
        <omgdi:waypoint x="566.0" y="142.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-85DAA48B-046E-4884-8C5F-8FC39FE6C515" id="BPMNEdge_sid-85DAA48B-046E-4884-8C5F-8FC39FE6C515">
        <omgdi:waypoint x="240.0" y="142.0"></omgdi:waypoint>
        <omgdi:waypoint x="315.0" y="142.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```



发起流程：

![image-20200907113213428](/images/activiti6-20/image-20200907113213428.png)

![image-20200907113351688](/images/activiti6-20/image-20200907113351688.png)

部门经理获取待办：发现没有任何待办，我们等5分钟后再次获取待办

![image-20200907113059113](/images/activiti6-20/image-20200907113059113.png)

等待5分钟，再次查看可以看到已经产生了一条待办

![image-20200907113726343](/images/activiti6-20/image-20200907113726343.png)

## 2、信号中间事件

信号中间事件分为2种：**中间信号捕获事件**、**中间信号抛出事件**

### 2.1 中间事件抛出事件

这个我们在介绍信号边界事件时已经介绍过了，这里不再赘述。

### 2.2 中间信号捕获事件

此处我们画一个简单的流程图：流程发起后，到达信号捕获中间事件，等待被触发。

![image-20200907143051031](/images/activiti6-20/image-20200907143051031.png)

bpmn流程定义：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <signal id="signal002" name="signal002" activiti:scope="global"></signal>
  <process id="intermediateSignalCaptureEvent" name="中间信号捕获事件" isExecutable="true">
    <startEvent id="sid-F3F76546-2B7A-4A2B-8F75-3D3C5A953FF0"></startEvent>
    <intermediateCatchEvent id="sid-8EF1CE6D-9F8C-4245-ABC5-CBEFA76A61CD" name="信号捕获">
      <signalEventDefinition signalRef="signal002"></signalEventDefinition>
    </intermediateCatchEvent>
    <sequenceFlow id="sid-E9DE460A-F4B3-47D7-9E55-80BB74C94175" sourceRef="sid-F3F76546-2B7A-4A2B-8F75-3D3C5A953FF0" targetRef="sid-8EF1CE6D-9F8C-4245-ABC5-CBEFA76A61CD"></sequenceFlow>
    <userTask id="sid-7F4109CB-1E68-48AC-93B4-CE4413CCAD31" name="部门经理审批" activiti:candidateGroups="deptLeader"></userTask>
    <sequenceFlow id="sid-016CB55A-B8AA-4CB8-B21C-31AD7C8391C0" sourceRef="sid-8EF1CE6D-9F8C-4245-ABC5-CBEFA76A61CD" targetRef="sid-7F4109CB-1E68-48AC-93B4-CE4413CCAD31"></sequenceFlow>
    <endEvent id="sid-4C1CE785-A3B5-4E5F-9347-6426E6FAE1A5"></endEvent>
    <sequenceFlow id="sid-FFDF46BF-B4BA-43DF-9DFE-E3AE6307B205" sourceRef="sid-7F4109CB-1E68-48AC-93B4-CE4413CCAD31" targetRef="sid-4C1CE785-A3B5-4E5F-9347-6426E6FAE1A5"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_intermediateSignalCaptureEvent">
    <bpmndi:BPMNPlane bpmnElement="intermediateSignalCaptureEvent" id="BPMNPlane_intermediateSignalCaptureEvent">
      <bpmndi:BPMNShape bpmnElement="sid-F3F76546-2B7A-4A2B-8F75-3D3C5A953FF0" id="BPMNShape_sid-F3F76546-2B7A-4A2B-8F75-3D3C5A953FF0">
        <omgdc:Bounds height="30.0" width="30.0" x="165.0" y="90.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-8EF1CE6D-9F8C-4245-ABC5-CBEFA76A61CD" id="BPMNShape_sid-8EF1CE6D-9F8C-4245-ABC5-CBEFA76A61CD">
        <omgdc:Bounds height="30.0" width="30.0" x="270.0" y="90.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-7F4109CB-1E68-48AC-93B4-CE4413CCAD31" id="BPMNShape_sid-7F4109CB-1E68-48AC-93B4-CE4413CCAD31">
        <omgdc:Bounds height="80.0" width="100.0" x="375.0" y="65.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-4C1CE785-A3B5-4E5F-9347-6426E6FAE1A5" id="BPMNShape_sid-4C1CE785-A3B5-4E5F-9347-6426E6FAE1A5">
        <omgdc:Bounds height="28.0" width="28.0" x="520.0" y="91.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-016CB55A-B8AA-4CB8-B21C-31AD7C8391C0" id="BPMNEdge_sid-016CB55A-B8AA-4CB8-B21C-31AD7C8391C0">
        <omgdi:waypoint x="300.0" y="105.0"></omgdi:waypoint>
        <omgdi:waypoint x="375.0" y="105.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-E9DE460A-F4B3-47D7-9E55-80BB74C94175" id="BPMNEdge_sid-E9DE460A-F4B3-47D7-9E55-80BB74C94175">
        <omgdi:waypoint x="195.0" y="105.0"></omgdi:waypoint>
        <omgdi:waypoint x="270.0" y="105.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-FFDF46BF-B4BA-43DF-9DFE-E3AE6307B205" id="BPMNEdge_sid-FFDF46BF-B4BA-43DF-9DFE-E3AE6307B205">
        <omgdi:waypoint x="475.0" y="105.0"></omgdi:waypoint>
        <omgdi:waypoint x="520.0" y="105.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

流程启动：

![image-20200907143627448](/images/activiti6-20/image-20200907143627448.png)

![image-20200907143713526](/images/activiti6-20/image-20200907143713526.png)

触发信号：之前我们已经有了信号触发接口：

![image-20200907144820388](/images/activiti6-20/image-20200907144820388.png)

不过此处需要**executionId**和**signalName** 参数，其中signalName在流程定义中可以看到：signal002。我們先获取**executionId**：

![image-20200907145036252](/images/activiti6-20/image-20200907145036252.png)

我们已经获取到了executionId为177510。接下来触发信号：

![image-20200907145210065](/images/activiti6-20/image-20200907145210065.png)

此时查看流程图如下：

![image-20200907145239710](/images/activiti6-20/image-20200907145239710.png)

而且，部门经理的待办也产生了，符合我们的预期：

![image-20200907145405803](/images/activiti6-20/image-20200907145405803.png)

## 3、中间消息捕获事件

中间消息捕获事件和中间信号捕获事件类似，我们也画一个简单流程图：

![image-20200907150948350](/images/activiti6-20/image-20200907150948350.png)

bpmn流程定义：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <message id="message002" name="message002"></message>
  <process id="intermediateMessageCaptureEvent" name="中间消息捕获事件" isExecutable="true">
    <startEvent id="sid-FBD3D904-2229-4A2B-8A54-0E95702C3BC5"></startEvent>
    <intermediateCatchEvent id="sid-0820F309-666F-470A-9728-6F52F8B983AF">
      <messageEventDefinition messageRef="message002"></messageEventDefinition>
    </intermediateCatchEvent>
    <userTask id="sid-B9B71A31-9EAA-4EAB-843A-AF152AA6B632" name="部门经理" activiti:candidateGroups="deptLeader"></userTask>
    <sequenceFlow id="sid-AF8D16CD-965A-4E7A-879D-F4FF6D4B1821" sourceRef="sid-0820F309-666F-470A-9728-6F52F8B983AF" targetRef="sid-B9B71A31-9EAA-4EAB-843A-AF152AA6B632"></sequenceFlow>
    <endEvent id="sid-E8A017FC-35CE-4BFB-A7D5-6F6A6866D849"></endEvent>
    <sequenceFlow id="sid-8308727F-04A4-43CE-8E4C-D0EDC84ACA5B" sourceRef="sid-B9B71A31-9EAA-4EAB-843A-AF152AA6B632" targetRef="sid-E8A017FC-35CE-4BFB-A7D5-6F6A6866D849"></sequenceFlow>
    <sequenceFlow id="sid-125B2FFF-2CA2-4837-8149-751E38FE0A12" sourceRef="sid-FBD3D904-2229-4A2B-8A54-0E95702C3BC5" targetRef="sid-0820F309-666F-470A-9728-6F52F8B983AF"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_intermediateMessageCaptureEvent">
    <bpmndi:BPMNPlane bpmnElement="intermediateMessageCaptureEvent" id="BPMNPlane_intermediateMessageCaptureEvent">
      <bpmndi:BPMNShape bpmnElement="sid-FBD3D904-2229-4A2B-8A54-0E95702C3BC5" id="BPMNShape_sid-FBD3D904-2229-4A2B-8A54-0E95702C3BC5">
        <omgdc:Bounds height="30.0" width="30.0" x="165.0" y="209.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-0820F309-666F-470A-9728-6F52F8B983AF" id="BPMNShape_sid-0820F309-666F-470A-9728-6F52F8B983AF">
        <omgdc:Bounds height="30.0" width="30.0" x="285.0" y="209.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-B9B71A31-9EAA-4EAB-843A-AF152AA6B632" id="BPMNShape_sid-B9B71A31-9EAA-4EAB-843A-AF152AA6B632">
        <omgdc:Bounds height="80.0" width="100.0" x="375.0" y="184.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-E8A017FC-35CE-4BFB-A7D5-6F6A6866D849" id="BPMNShape_sid-E8A017FC-35CE-4BFB-A7D5-6F6A6866D849">
        <omgdc:Bounds height="28.0" width="28.0" x="540.0" y="210.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-8308727F-04A4-43CE-8E4C-D0EDC84ACA5B" id="BPMNEdge_sid-8308727F-04A4-43CE-8E4C-D0EDC84ACA5B">
        <omgdi:waypoint x="475.0" y="224.0"></omgdi:waypoint>
        <omgdi:waypoint x="540.0" y="224.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-AF8D16CD-965A-4E7A-879D-F4FF6D4B1821" id="BPMNEdge_sid-AF8D16CD-965A-4E7A-879D-F4FF6D4B1821">
        <omgdi:waypoint x="315.0" y="224.0"></omgdi:waypoint>
        <omgdi:waypoint x="375.0" y="224.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-125B2FFF-2CA2-4837-8149-751E38FE0A12" id="BPMNEdge_sid-125B2FFF-2CA2-4837-8149-751E38FE0A12">
        <omgdi:waypoint x="195.0" y="224.0"></omgdi:waypoint>
        <omgdi:waypoint x="285.0" y="224.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```



发起流程：

![image-20200907150938887](/images/activiti6-20/image-20200907150938887.png)

![image-20200907151056206](/images/activiti6-20/image-20200907151056206.png)

此处和信号中间事件一样，也需要获取executionId（180013），messageName流程定义中已经说明：message002

![image-20200907151232970](/images/activiti6-20/image-20200907151232970.png)

触发消息：

![image-20200907151404216](/images/activiti6-20/image-20200907151404216.png)

![image-20200907151532678](/images/activiti6-20/image-20200907151532678.png)

可以看到消息已经被触发。符合预期。