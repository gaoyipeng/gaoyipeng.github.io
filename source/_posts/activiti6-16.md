---
title: Activiti（16）Activiti事件
date: 2020-08-16 17:08:25
tags: Activiti
categories: Activiti
description: Activiti事件
typora-root-url: ..
password: kiki

---

本节我们介绍Activiti的事件类型。

## 1、启动事件

从`Activiti Modeler`中即可看到，启动事件有以下这么几种，我们前面流程中使用的都是普通的开始事件。接下来我们介绍其他常用的开始事件。

![image-20200816171238387](/images/activiti6-16/image-20200816171238387.png)

### 1.1 定时启动事件

定时器启动事件用于在给定的时间点创建流程实例。它可以用在只启动一次的流程中，也可以用在特定时间间隔下启动。

**注意：**子流程中不能使用定时器启动事件。 **定时器是从流程部署开始计时，不需要去启动流程。**

#### 1.1.1 定时器分类

- ![image-20200816184632442](/images/activiti6-16/image-20200816184632442.png)**timeDate**

  使用 ISO 8601 格式指定一个确定的时间点。

  在框中输入2020-08-16T12:13:14代表2020年08月16号，12点13分14秒这个时间点执行此定时器。

  ```
  <timerEventDefinition>
      <timeDate>2020-08-16T12:13:14</timeDate>
  </timerEventDefinition>
  ```

- **timeDuration**

  指定定时器之前要等待多长时间。 使用ISO 8601规定的格式 （由BPMN 2.0规定）

  ```
  
  ## P1D：代表1天后执行此时间定时器。
  ## P1H：代表1小时后执行此时间定时器。
  ## P1M：代表1分钟后执行此时间定时器。
  ## PT1M：代表1分钟后执行此时间定时器。
  
  <timerEventDefinition>
      <timeDuration>P10M</timeDuration>
  </timerEventDefinition>
  ```

- **timeCycle**

  指定重复执行的间隔， 可以用来定期启动流程实例。 timeCycle元素可以使用两种格式。

  - 第一种是 ISO 8601 标准的格式。示例：（重复3次，每次间隔10小时）：

  ```
  <timerEventDefinition>
      <timeCycle>R3/PT10H</timeCycle>
  </timerEventDefinition>
  ```

  - 第二种是使用cron表达式指定timeCycle。示例：（每分钟执行一次）

  ```
  <timerEventDefinition>
  	<timeCycle>0 0/1 * * * ?</timeCycle>
  </timerEventDefinition>
  ```

  cron表达式生成器：[https://cron.qqe2.com/](https://cron.qqe2.com/)

#### 1.1.2 示例

此处我们画一个简单的流程来验证一下：

流程很简单，每隔一分钟启动此流程，流转到**部门领导办理**节点，然后结束。

![image-20200816174807017](/images/activiti6-16/image-20200816174807017.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <process id="timing-start-event" name="定时开始事件" isExecutable="true">
    <documentation>用于测试定时开始事件</documentation>
    <startEvent id="startEvent" activiti:isInterrupting="false">
      <documentation>每隔一分钟发起一个流程</documentation>
      <timerEventDefinition>
        <timeCycle>0 0/1 * * * ?</timeCycle>
      </timerEventDefinition>
    </startEvent>
    <userTask id="manageApprove" name="部门领导办理" activiti:candidateGroups="deptLeader"></userTask>
    <sequenceFlow id="sid-A74B2996-9B0B-4791-97EE-739BA5284496" sourceRef="startEvent" targetRef="manageApprove"></sequenceFlow>
    <endEvent id="sid-35EBDCF2-C4A3-4BA2-B5C7-DA216ED2C7BB"></endEvent>
    <sequenceFlow id="sid-F4C07822-864B-421C-9506-23EAC7EAD893" sourceRef="manageApprove" targetRef="sid-35EBDCF2-C4A3-4BA2-B5C7-DA216ED2C7BB"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_timing-start-event">
    <bpmndi:BPMNPlane bpmnElement="timing-start-event" id="BPMNPlane_timing-start-event">
      <bpmndi:BPMNShape bpmnElement="startEvent" id="BPMNShape_startEvent">
        <omgdc:Bounds height="31.0" width="31.0" x="156.0" y="230.5"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="manageApprove" id="BPMNShape_manageApprove">
        <omgdc:Bounds height="80.0" width="100.0" x="232.0" y="206.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-35EBDCF2-C4A3-4BA2-B5C7-DA216ED2C7BB" id="BPMNShape_sid-35EBDCF2-C4A3-4BA2-B5C7-DA216ED2C7BB">
        <omgdc:Bounds height="28.0" width="28.0" x="377.0" y="232.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-F4C07822-864B-421C-9506-23EAC7EAD893" id="BPMNEdge_sid-F4C07822-864B-421C-9506-23EAC7EAD893">
        <omgdi:waypoint x="332.0" y="246.0"></omgdi:waypoint>
        <omgdi:waypoint x="377.0" y="246.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-A74B2996-9B0B-4791-97EE-739BA5284496" id="BPMNEdge_sid-A74B2996-9B0B-4791-97EE-739BA5284496">
        <omgdi:waypoint x="187.99983471330506" y="246.42727347857587"></omgdi:waypoint>
        <omgdi:waypoint x="232.0" y="246.22727272727272"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

部署流程什么的我就不再演示了，相信大家都很熟悉了。

我们直接查看部门领导`leaderuser`的当前待办。

![image-20200816175133619](/images/activiti6-16/image-20200816175133619.png)

返回值：

```json
{
    "code":"0000",
    "data":{
        "pageNum":1,
        "pageSize":10,
        "total":13,
        "pages":0,
        "firstResult":0,
        "maxResults":10,
        "list":[
            {
                "owner":null,
                "originalAssignee":null,
                "assignee":null,
                "delegationState":null,
                "parentTaskId":null,
                "name":"部门领导办理",
                "localizedName":null,
                "description":null,
                "localizedDescription":null,
                "priority":50,
                "createTime":"2020-08-16 09:51:02",
                "dueDate":null,
                "suspensionState":1,
                "category":null,
                "executionId":"7558",
                "processInstanceId":"7557",
                "processDefinitionId":"timing-start-event:1:5004",
                "taskDefinitionKey":"manageApprove",
                "formKey":null,
                "isDeleted":false,
                "isCanceled":false,
                "eventName":null,
                "currentActivitiListener":null,
                "tenantId":"",
                "queryVariables":null,
                "claimTime":null,
                "usedVariablesCache":{

                },
                "cachedElContext":null,
                "id":"7562",
                "revision":1,
                "isInserted":false,
                "isUpdated":false
            },
            Object{...},
            Object{...},
            Object{...},
            Object{...},
            Object{...},
            Object{...},
            Object{...},
            Object{...},
            Object{...}
        ]
    },
    "message":"获取当前人员待办列表成功"
}
```

可以看到已经自动生成了很多待办。