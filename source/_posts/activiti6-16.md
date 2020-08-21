---
title: Activiti（16）Activiti启动事件
date: 2020-08-16 17:08:25
tags: Activiti
categories: Activiti
description: Activiti启动事件
typora-root-url: ..
password: kiki
---

本节我们介绍Activiti的启动事件类型。

# 1、启动事件

从`Activiti Modeler`中即可看到，启动事件有以下这么几种，我们前面流程中使用的都是普通的开始事件。接下来我们介绍其他常用的开始事件。

![image-20200816171238387](/images/activiti6-16/image-20200816171238387.png)

## 1.1 开始事件

“空”启动事件，指的是没有特别指定启动流程实例的触发器。这意味着引擎无法预知何时启动流程实例。可以通过以下方法启动：

```java
ProcessInstance processInstance = runtimeService.startProcessInstanceByXXX();
formService.submitStartFormData();
```

**注意：**子流程（subprocess）总是有空启动事件。

空启动事件的XML表示格式，就是普通的启动事件声明，而没有任何子元素（其他种类的启动事件都有子元素，用于声明其类型）。

```xml
<startEvent id="start" name="my start event" />
```

前面我们已经见过很多了，这里不做演示了。

## 1.2 定时启动事件

定时器启动事件用于在给定的时间点创建流程实例。它可以用在只启动一次的流程中，也可以用在特定时间间隔下启动。

XML表示：

```xml
<startEvent id="theStart">
    <timerEventDefinition>
        <timeDate>2020-08-16T12:13:14</timeDate>
    </timerEventDefinition>
</startEvent>
```

**注意：**

1、子流程中不能使用定时器启动事件。 **定时器是从流程部署开始计时，不需要去启动流程。**

2、当部署带有定时器启动事件的流程的新版本时，上一版本的定时器作业会被移除。这是因为通常并不希望旧版本的流程仍然自动启动新的流程实例。

### 1.1.1 定时器分类

- ![image-20200816184632442](/images/activiti6-16/image-20200816184632442.png)

  

- **timeDate**：一次性启动

  使用 ISO 8601 格式指定一个确定的时间点。

  在框中输入2020-08-16T12:13:14代表2020年08月16号，12点13分14秒这个时间点执行此定时器。

  ```
  <startEvent id="theStart">
      <timerEventDefinition>
          <timeDate>2020-08-16T12:13:14</timeDate>
      </timerEventDefinition>
  </startEvent>
  ```

- **timeDuration**：设置多长时间启动

  指定定时器之前要等待多长时间。 使用ISO 8601规定的格式 （由BPMN 2.0规定）

  ```
  
  ## P1D：代表1天后执行此时间定时器。
  ## P1H：代表1小时后执行此时间定时器。
  ## P1M：代表1分钟后执行此时间定时器。
  ## PT1M：代表1分钟后执行此时间定时器。
  <startEvent id="theStart">
      <timerEventDefinition>
          <timeDuration>P10M</timeDuration>
      </timerEventDefinition>
  </startEvent>
  ```

- **timeCycle**：周期性循环启动

  指定重复执行的间隔， 可以用来定期启动流程实例。 timeCycle元素可以使用两种格式。

  - 第一种是 ISO 8601 标准的格式。示例：（重复3次，每次间隔10小时）：

  ```
  <startEvent id="theStart">
      <timerEventDefinition>
          <timeCycle>R3/PT10H</timeCycle>
      </timerEventDefinition>
</startEvent>
  ```

  - 第二种是使用cron表达式指定timeCycle。示例：（每分钟执行一次）
  
  ```
  <startEvent id="theStart">
      <timerEventDefinition>
        <timeCycle>0 0/1 * * * ?</timeCycle>
      </timerEventDefinition>
  <startEvent id="theStart">
  ```
  
  cron表达式生成器：[https://cron.qqe2.com/](https://cron.qqe2.com/)

### 1.1.2 示例

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

## 1.3 消息开始事件

> 消息启动事件，使用具名消息启动流程实例。它让我们可以使用消息名，有效地在一组可选的启动事件中*选择*正确的启动事件。

上面是官方的介绍，看着拗口，说白了就是通过一个消息ID=xxx，将已部署的流程定义中启动事件是消息启动事件的，并且监听ID的消息为xx的流程启动。

XML 表示：

```xml
<message id="message-start-01" name="message-start-01"></message>
<startEvent id="message-start" name="消息启动" activiti:isInterrupting="false">
      <messageEventDefinition messageRef="message-start-01"></messageEventDefinition>
</startEvent>
```

### 1.2.1 示例

此处我们依然画一个简单流程图来验证。

![image-20200819182710738](/images/activiti6-16/image-20200819182710738.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <message id="message-start-01" name="message-start-01"></message>
  <process id="message-start-event" name="消息启动事件" isExecutable="true">
    <documentation>测试消息启动事件</documentation>
    <userTask id="manageApprove" name="部门领导审批" activiti:candidateGroups="deptLeader"></userTask>
    <endEvent id="sid-94160C92-3C13-4EC1-8182-771B59FE6497"></endEvent>
    <sequenceFlow id="sid-5113EC9A-BC30-4804-9177-179C6DA065A8" sourceRef="manageApprove" targetRef="sid-94160C92-3C13-4EC1-8182-771B59FE6497"></sequenceFlow>
    <startEvent id="message-start" name="消息启动" activiti:isInterrupting="false">
      <messageEventDefinition messageRef="message-start-01"></messageEventDefinition>
    </startEvent>
    <sequenceFlow id="sid-BA8C8F05-153A-4A47-9E47-ED54DC04A740" sourceRef="message-start" targetRef="manageApprove"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_message-start-event">
    <bpmndi:BPMNPlane bpmnElement="message-start-event" id="BPMNPlane_message-start-event">
      <bpmndi:BPMNShape bpmnElement="manageApprove" id="BPMNShape_manageApprove">
        <omgdc:Bounds height="80.0" width="100.0" x="255.0" y="166.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-94160C92-3C13-4EC1-8182-771B59FE6497" id="BPMNShape_sid-94160C92-3C13-4EC1-8182-771B59FE6497">
        <omgdc:Bounds height="28.0" width="28.0" x="405.0" y="192.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="message-start" id="BPMNShape_message-start">
        <omgdc:Bounds height="30.0" width="30.5" x="150.0" y="191.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-BA8C8F05-153A-4A47-9E47-ED54DC04A740" id="BPMNEdge_sid-BA8C8F05-153A-4A47-9E47-ED54DC04A740">
        <omgdi:waypoint x="205.79998779296875" y="206.0"></omgdi:waypoint>
        <omgdi:waypoint x="255.0" y="206.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-5113EC9A-BC30-4804-9177-179C6DA065A8" id="BPMNEdge_sid-5113EC9A-BC30-4804-9177-179C6DA065A8">
        <omgdi:waypoint x="355.0" y="206.0"></omgdi:waypoint>
        <omgdi:waypoint x="405.0" y="206.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```



- 定义消息

  点击流程图空白处，先定义一个消息。消息id：message-start-01

  ![image-20200819182939708](/images/activiti6-16/image-20200819182939708.png)

  ![image-20200819183000998](/images/activiti6-16/image-20200819183000998.png)

- 设置消息启动事件

  在启动节点选择上一步定义好的消息。

  ![image-20200819183120975](/images/activiti6-16/image-20200819183120975.png)

  其他节点就不做介绍了。

  

### 1.2.2  启动方式

消息开始事件的发起，和普通开始事件是不同的。我们来看一下迄今为止，我们用到过的流程启动方法：

```java
 formService.submitStartFormData(processDefinitionId, formProperties);
 runtimeService.startProcessInstanceByKey("leave", businessKey, variables);
```

而消息开始事件用的是以下的方法

```java
ProcessInstance startProcessInstanceByMessage(String messageName);
ProcessInstance startProcessInstanceByMessage(String messageName, String businessKey);
ProcessInstance startProcessInstanceByMessage(String messageName, Map<String, Object> processVariables);
ProcessInstance startProcessInstanceByMessage(String messageName, String businessKey, Map<String, Object< processVariables);
```



我们新建一个启动方法，参考`/process/startProcess/{processDefinitionId}`接口即可。

```java
/**
 * 消息启动流程
 */
@PostMapping(value = "/messageStartEventInstance/{messageId}")
@ApiOperation(value = "消息启动流程",notes = "发起启动节点是消息启动类型的流程，key需要以fp_开头")
public CommonResponse messageStartEventInstance(@PathVariable("messageId") @ApiParam("消息定义的编号")String messageId,
                                                HttpServletRequest request) throws CommonException {
    ProcessInstance processInstance = processService.messageStartEventInstance(messageId,request);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("流程启动成功，流程ID：" + processInstance.getId()).data(processInstance.getId());
}
```



```java
@Override
public ProcessInstance messageStartEventInstance(String message, HttpServletRequest request) throws CommonException {
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

    ProcessInstance processInstance = null;
    try {
        identityService.setAuthenticatedUserId(user.getId());
        processInstance = runtimeService.startProcessInstanceByMessage(message,formProperties);

        log.debug("start a processinstance: {}", processInstance);
    } finally {
        identityService.setAuthenticatedUserId(null);
    }
    return processInstance;
}
```

### 1.2.3 测试

![image-20200819185928285](/images/activiti6-16/image-20200819185928285.png)

![image-20200819190119903](/images/activiti6-16/image-20200819190119903.png)

可以看到，部门领导用户已经有了这条流程的待办了。

### 1.2.4 注意事项

当**部署**具有一个或多个消息启动事件的流程定义时，需要注意：

- 消息启动事件的名字，在给定流程定义中，必须是唯一的。一个流程定义不得包含多个同名的消息启动事件。如果流程定义中有两个或多个消息启动事件引用 同一个消息，也即两个或多个消息启动事件引用了具有相同消息名字的消息，则Activiti在部署这个流程定义时，会抛出异常。
- 消息启动事件的名字，在所有已部署的流程定义中，必须是唯一的。如果流程定义中，一个或多个消息启动事件，引用了已经部署的另一流程定义中消息启动事件的消息名，则Activiti在部署这个流程定义时，会抛出异常。
- 流程版本：在部署流程定义的新版本时，会取消上一版本的消息订阅。即使新版本中并没有这个消息事件，仍然如此（取消上版本的消息订阅）。

## 1.4错误开始事件

 错误开始事件，可用于触发事件子流程（Event Sub-Process）。**错误启动事件不能用于启动流程实例**。

错误开始事件，往往和错误结束事件一起出现，可以是N对1的关系。异常结束事件抛出一个异常编码，只要和异常启动事件设置的异常编码匹配，即可开启事件子流程。

XML表示:

```xml
<!--错误结束-->
<endEvent id="errorendevent1" name="TerminateEndEvent">
    <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
</endEvent>
<!--错误开始-->
<startEvent id="messageStart" >
	<errorEventDefinition errorRef="PAYMENT_REJECT" />
</startEvent>
```

在前面的事件子流程章节，我们已经见过了，这里就不再赘述：

![image-20200820104351195](/images/activiti6-16/image-20200820104351195.png)

## 1.5 信号开始事件

> 信号启动事件，使用具名信号启动流程实例。这个信号可以由流程实例中的信号抛出中间事件（intermediary signal throw event），或者`API`（`runtimeService.signalEventReceivedXXX`方法）触发。这些情况下，所有拥有相同名字信号启动事件的流程定义都会被启动。

### 1.5.1 示例

我们创建一个简单流程

![image-20200820145210752](/images/activiti6-16/image-20200820145210752.png)

设置信号定义：

![image-20200820152733416](/images/activiti6-16/image-20200820152733416.png)

![image-20200820153723808](/images/activiti6-16/image-20200820153723808.png)

Scope说明：

- `global`:全局
- `processInstance`:当前流程

默认信号会在流程引擎范围内进行广播。就是说， 你可以在一个流程实例中抛出一个信号事件，其他不同流程定义的流程实例 都可以监听到这个事件。然而，有时只希望在同一个流程实例中响应这个信号事件。

![image-20200820153815439](/images/activiti6-16/image-20200820153815439.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <signal id="signal-start-01" name="signal-start-01" activiti:scope="processInstance"></signal>
  <signal id="signal-start-02" name="signal-start-02" activiti:scope="global"></signal>
  <process id="signal-start-event" name="信号开始事件" isExecutable="true">
    <startEvent id="signal-start" name="信号启动" activiti:isInterrupting="false">
      <signalEventDefinition signalRef="signal-start-01"></signalEventDefinition>
    </startEvent>
    <userTask id="manageApprove" name="部门领导审批" activiti:candidateGroups="deptLeader"></userTask>
    <sequenceFlow id="sid-A3B56AF2-FEE1-49DB-850D-A7DDB2C11C4B" sourceRef="signal-start" targetRef="manageApprove"></sequenceFlow>
    <endEvent id="sid-908769D7-DF1B-471F-9BDB-0CCEC7EB0D18"></endEvent>
    <sequenceFlow id="sid-EBB4CE5E-311A-4BAE-B367-0FF462B103C1" sourceRef="manageApprove" targetRef="sid-908769D7-DF1B-471F-9BDB-0CCEC7EB0D18"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_signal-start-event">
    <bpmndi:BPMNPlane bpmnElement="signal-start-event" id="BPMNPlane_signal-start-event">
      <bpmndi:BPMNShape bpmnElement="signal-start" id="BPMNShape_signal-start">
        <omgdc:Bounds height="30.0" width="30.0" x="191.39999999999998" y="273.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="manageApprove" id="BPMNShape_manageApprove">
        <omgdc:Bounds height="80.0" width="100.0" x="266.4" y="248.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-908769D7-DF1B-471F-9BDB-0CCEC7EB0D18" id="BPMNShape_sid-908769D7-DF1B-471F-9BDB-0CCEC7EB0D18">
        <omgdc:Bounds height="28.0" width="28.0" x="411.4" y="274.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-EBB4CE5E-311A-4BAE-B367-0FF462B103C1" id="BPMNEdge_sid-EBB4CE5E-311A-4BAE-B367-0FF462B103C1">
        <omgdi:waypoint x="366.4" y="288.0"></omgdi:waypoint>
        <omgdi:waypoint x="411.4" y="288.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-A3B56AF2-FEE1-49DB-850D-A7DDB2C11C4B" id="BPMNEdge_sid-A3B56AF2-FEE1-49DB-850D-A7DDB2C11C4B">
        <omgdi:waypoint x="221.39999999999998" y="288.0"></omgdi:waypoint>
        <omgdi:waypoint x="266.4" y="288.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

### 1.5.2  启动方式

- `API`主动触发：调用 `runtimeService.signalEventReceivedXXX`方法,可以看到提供了异步启动API

  ![image-20200821104915953](/images/activiti6-16/image-20200821104915953.png)

- 由流程实例中的信号抛出中间事件触发

  这个我们后面修学习到信号抛出中间事件再介绍。

我们新建一个启动方法

```java
/**
 * 信号启动流程
 */
@PostMapping(value = "/signalStartEventInstance/{signalId}")
@ApiOperation(value = "信号启动流程",notes = "发起启动节点是信号启动类型的流程，key需要以fp_开头")
public CommonResponse signalStartEventInstance(@PathVariable("signalId") @ApiParam("信号定义的编号")String signalId,
                                               HttpServletRequest request) throws CommonException {
    processService.signalStartEventInstance(signalId,request);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("信号启动流程成功");
}
```

```java
/**
* 信号启动流程
* @param signalId
* @param request
* @return
* @throws CommonException
*/
@Override
public void signalStartEventInstance(String signalId, HttpServletRequest request) throws CommonException {
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

    ProcessInstance processInstance = null;
    try {
        identityService.setAuthenticatedUserId(user.getId());
        runtimeService.signalEventReceived(signalId,formProperties);
    } finally {
        identityService.setAuthenticatedUserId(null);
    }
}
```

### 1.5.3 测试

部署流程后开始测试。

![image-20200821110616766](/images/activiti6-16/image-20200821110616766.png)

使用leaderuser启动系统，获取待办

![image-20200821110642761](/images/activiti6-16/image-20200821110642761.png)

![image-20200821111436096](/images/activiti6-16/image-20200821111436096.png)

可以看到已经获取了待办。

