---
title: Activiti（12） 事件子流程
date: 2020-07-15 09:05:58
tags: Activiti
categories: Activiti
description: Activiti（12） 事件子流程
typora-root-url: ..
password: kiki
---

> **事件子流程**：它和子流程类似，把一系列的活动归结到一起处理，不同的是事件子流程不能直接启动，而要“被动”的由其它事件触发启动。

# 1、事件子流程

![image-20200715103931544](/images/activiti6-13/image-20200715103931544.png)

在上一节子流程中，如果付费子流程未通过财务或总经理审批，则定义了一个**错误结束事件**，并抛出一个**异常编码**，会被附加在付费子流程上的**异常边界事件**捕获，将流程定义到调整申请节点。这是处理流程异常的一种方式。

而上图提供了另一种处理办法。不使用异常边界事件，而是定义一个子流程（包含一个事件启动流程，可以是异常启动事件、消息启动事件）来处理。图中异常结束事件抛出一个异常编码，只要和异常启动事件设置的异常编码匹配，即可开启 子流程。

## 1.1 流程定义

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.kafeitu.me/activiti">
  <process id="purchase-error-event-subprocess-kiki" name="办公用品采购--事件子流程(异常)" isExecutable="true">
    <startEvent id="startevent1" name="startevent1" activiti:initiator="applyUserId">
      <extensionElements>
        <activiti:formProperty id="dueDate" name="到货期限" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <activiti:formProperty id="listing" name="物品清单" type="bigtext" required="true"></activiti:formProperty>
        <activiti:formProperty id="amountMoney" name="总金额" type="double" required="true"></activiti:formProperty>
      </extensionElements>
    </startEvent>
    <userTask id="deptLeaderAudit" name="领导审批" activiti:candidateGroups="deptLeader">
      <extensionElements>
        <activiti:formProperty id="applyUser" name="申请人" type="string" variable="applyUserId" writable="false"></activiti:formProperty>
        <activiti:formProperty id="dueDate" name="到货期限" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
        <activiti:formProperty id="listing" name="物品清单" type="bigtext" writable="false"></activiti:formProperty>
        <activiti:formProperty id="amountMoney" name="总金额" type="double" writable="false"></activiti:formProperty>
        <activiti:formProperty id="deptLeaderApproved" name="是否同意" type="enum">
          <activiti:value id="true" name="同意"></activiti:value>
          <activiti:value id="false" name="驳回"></activiti:value>
        </activiti:formProperty>
      </extensionElements>
    </userTask>
    <exclusiveGateway id="exclusivegateway1" name="Exclusive Gateway"></exclusiveGateway>
    <sequenceFlow id="flow2" sourceRef="deptLeaderAudit" targetRef="exclusivegateway1"></sequenceFlow>
    <userTask id="contactSupplier" name="联系供货方" activiti:candidateGroups="supportCrew">
      <extensionElements>
        <activiti:formProperty id="applyUser" name="申请人" type="string" variable="applyUserId" writable="false"></activiti:formProperty>
        <activiti:formProperty id="dueDate" name="到货期限" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
        <activiti:formProperty id="listing" name="物品清单" type="bigtext" writable="false"></activiti:formProperty>
        <activiti:formProperty id="amountMoney" name="总金额" type="double" writable="false"></activiti:formProperty>
        <activiti:formProperty id="supplier" name="供货方" type="string" required="true"></activiti:formProperty>
        <activiti:formProperty id="bankName" name="开户行" type="string" required="true"></activiti:formProperty>
        <activiti:formProperty id="bankAccount" name="银行账号" type="string" required="true"></activiti:formProperty>
        <activiti:formProperty id="planDate" name="预计交货日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
      </extensionElements>
    </userTask>
    <sequenceFlow id="flow3" sourceRef="exclusivegateway1" targetRef="contactSupplier">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderApproved == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <subProcess id="subprocessPay" name="付费子流程">
      <startEvent id="startevent2" name="Start"></startEvent>
      <exclusiveGateway id="exclusivegateway2" name="Exclusive Gateway"></exclusiveGateway>
      <sequenceFlow id="flow5" sourceRef="treasurerAudit" targetRef="exclusivegateway-treasurerAudit"></sequenceFlow>
      <userTask id="generalManagerAudit" name="总经理审批" activiti:candidateGroups="generalManager">
        <extensionElements>
          <activiti:formProperty id="applyUser" name="申请人" type="string" variable="applyUserId" writable="false"></activiti:formProperty>
          <activiti:formProperty id="usage" name="用途" type="bigtext" expression="${usage}" writable="false"></activiti:formProperty>
          <activiti:formProperty id="amountMoney" name="总金额" type="double" writable="false"></activiti:formProperty>
          <activiti:formProperty id="generalManagerApproved" name="是否同意" type="enum">
            <activiti:value id="true" name="同意"></activiti:value>
            <activiti:value id="false" name="驳回"></activiti:value>
          </activiti:formProperty>
        </extensionElements>
      </userTask>
      <sequenceFlow id="flow6" name="金额 &gt;= 1万元" sourceRef="exclusivegateway2" targetRef="generalManagerAudit">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${amountMoney >= 10000}]]></conditionExpression>
      </sequenceFlow>
      <exclusiveGateway id="exclusivegateway-generalManagerAudit" name="Exclusive Gateway"></exclusiveGateway>
      <sequenceFlow id="flow7" sourceRef="generalManagerAudit" targetRef="exclusivegateway-generalManagerAudit"></sequenceFlow>
      <userTask id="pay" name="出纳付款" activiti:candidateGroups="cashier">
        <extensionElements>
          <activiti:formProperty id="applyUser" name="申请人" type="string" variable="applyUserId" writable="false"></activiti:formProperty>
          <activiti:formProperty id="usage" name="用途" type="bigtext" expression="${usage}" writable="false"></activiti:formProperty>
          <activiti:formProperty id="supplier" name="受款方" type="string" writable="false"></activiti:formProperty>
          <activiti:formProperty id="bankName" name="开户行" type="string" writable="false"></activiti:formProperty>
          <activiti:formProperty id="bankAccount" name="银行账号" type="string" writable="false"></activiti:formProperty>
        </extensionElements>
      </userTask>
      <sequenceFlow id="flow8" sourceRef="exclusivegateway-generalManagerAudit" targetRef="pay">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${generalManagerApproved == 'true'}]]></conditionExpression>
      </sequenceFlow>
      <endEvent id="endevent1" name="End"></endEvent>
      <sequenceFlow id="flow9" sourceRef="pay" targetRef="endevent1"></sequenceFlow>
      <sequenceFlow id="flow-generalManagerAudit" name="总经理不同意" sourceRef="exclusivegateway-generalManagerAudit" targetRef="errorendevent1">
        <extensionElements>
          <activiti:executionListener event="take" class="com.sxdx.workflow.activiti.rest.listener.HandleErrorInfoForPaymentListener"></activiti:executionListener>
        </extensionElements>
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${generalManagerApproved == 'false'}]]></conditionExpression>
      </sequenceFlow>
      <endEvent id="errorendevent1" name="TerminateEndEvent">
        <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
      </endEvent>
      <sequenceFlow id="flow18" name="金额 &lt; 1万" sourceRef="exclusivegateway2" targetRef="pay">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${amountMoney < 10000}]]></conditionExpression>
      </sequenceFlow>
      <userTask id="treasurerAudit" name="财务审批" activiti:candidateGroups="treasurer">
        <extensionElements>
          <activiti:formProperty id="applyUser" name="申请人" type="string" variable="applyUserId" writable="false"></activiti:formProperty>
          <activiti:formProperty id="usage" name="用途" type="bigtext" expression="${usage}" writable="false"></activiti:formProperty>
          <activiti:formProperty id="amountMoney" name="总金额" type="double" writable="false"></activiti:formProperty>
          <activiti:formProperty id="treasurerApproved" name="是否同意" type="enum">
            <activiti:value id="true" name="同意"></activiti:value>
            <activiti:value id="false" name="驳回"></activiti:value>
          </activiti:formProperty>
        </extensionElements>
      </userTask>
      <sequenceFlow id="flow25" sourceRef="startevent2" targetRef="treasurerAudit"></sequenceFlow>
      <exclusiveGateway id="exclusivegateway-treasurerAudit" name="Exclusive Gateway"></exclusiveGateway>
      <sequenceFlow id="flow26" sourceRef="exclusivegateway-treasurerAudit" targetRef="exclusivegateway2">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${treasurerApproved == 'true'}]]></conditionExpression>
      </sequenceFlow>
      <sequenceFlow id="flow-treasurerAudit" name="财务不同意" sourceRef="exclusivegateway-treasurerAudit" targetRef="errorendevent2">
        <extensionElements>
          <activiti:executionListener event="take" class="com.sxdx.workflow.activiti.rest.listener.HandleErrorInfoForPaymentListener"></activiti:executionListener>
        </extensionElements>
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${treasurerApproved == 'false'}]]></conditionExpression>
      </sequenceFlow>
      <endEvent id="errorendevent2" name="End">
        <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
      </endEvent>
    </subProcess>
    <sequenceFlow id="flow20" sourceRef="startevent1" targetRef="deptLeaderAudit"></sequenceFlow>
    <sequenceFlow id="flow22" name="进入付费子流程" sourceRef="contactSupplier" targetRef="subprocessPay">
      <extensionElements>
        <activiti:executionListener event="take" expression="${execution.setVariable('usage', listing)}"></activiti:executionListener>
      </extensionElements>
    </sequenceFlow>
    <userTask id="confirmReceipt" name="收货确认" activiti:assignee="${applyUserId}">
      <extensionElements>
        <activiti:formProperty id="applyUser" name="申请人" type="string" variable="applyUserId" writable="false"></activiti:formProperty>
        <activiti:formProperty id="dueDate" name="到货期限" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
        <activiti:formProperty id="listing" name="物品清单" type="bigtext" writable="false"></activiti:formProperty>
        <activiti:formProperty id="amountMoney" name="总金额" type="double" writable="false"></activiti:formProperty>
        <activiti:formProperty id="supplier" name="供货方" type="string" writable="false"></activiti:formProperty>
        <activiti:formProperty id="bankName" name="开户行" type="string" writable="false"></activiti:formProperty>
        <activiti:formProperty id="bankAccount" name="银行账号" type="string" writable="false"></activiti:formProperty>
        <activiti:formProperty id="planDate" name="预计交货日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
      </extensionElements>
    </userTask>
    <endEvent id="endevent3" name="End"></endEvent>
    <sequenceFlow id="flow23" sourceRef="confirmReceipt" targetRef="endevent3"></sequenceFlow>
    <sequenceFlow id="flow24" sourceRef="subprocessPay" targetRef="confirmReceipt"></sequenceFlow>
    <subProcess id="catchErrorForPayment" name="捕获付费子流程异常" triggeredByEvent="true">
      <startEvent id="errorstartevent1" name="Error start">
        <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
      </startEvent>
      <serviceTask id="recordErrorInfo" name="记录异常信息" activiti:expression="${execution.setVariable('ERROR_INFO', message)}"></serviceTask>
      <endEvent id="endevent6" name="End"></endEvent>
      <sequenceFlow id="flow29" sourceRef="recordErrorInfo" targetRef="endevent6"></sequenceFlow>
      <sequenceFlow id="flow34" sourceRef="errorstartevent1" targetRef="recordErrorInfo"></sequenceFlow>
    </subProcess>
    <endEvent id="endevent5" name="End"></endEvent>
    <userTask id="modifyApply" name="调整申请">
      <extensionElements>
        <activiti:formProperty id="dueDate" name="到货期限" type="date" expression="${dueDate}" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <activiti:formProperty id="listing" name="物品清单" type="bigtext" expression="${listing}" required="true"></activiti:formProperty>
        <activiti:formProperty id="amountMoney" name="总金额" type="double" expression="${amountMoney}" required="true"></activiti:formProperty>
      </extensionElements>
    </userTask>
    <sequenceFlow id="flow30" sourceRef="exclusivegateway1" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderApproved == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <exclusiveGateway id="exclusivegateway6" name="Exclusive Gateway"></exclusiveGateway>
    <sequenceFlow id="flow31" sourceRef="modifyApply" targetRef="exclusivegateway6"></sequenceFlow>
    <sequenceFlow id="flow32" name="调整后重新申请" sourceRef="exclusivegateway6" targetRef="deptLeaderAudit">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow33" sourceRef="exclusivegateway6" targetRef="endevent5">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'false'}]]></conditionExpression>
    </sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_purchase-error-event-subprocess">
    <bpmndi:BPMNPlane bpmnElement="purchase-error-event-subprocess" id="BPMNPlane_purchase-error-event-subprocess">
      //位置信息省略
       
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

## 1.2 财务审批

这个流程和上一节的办公用品采购很相似，人员配置也是一样的。

![image-20200715144724249](/images/activiti6-13/image-20200715144724249.png)

我们省略前面的环节，直接展示财务审批和总经理审批不通过时如何处理以及如何在事件子流程捕获异常。

```xml
<userTask id="treasurerAudit" name="财务审批" activiti:candidateGroups="treasurer">
    <extensionElements>
        <activiti:formProperty id="applyUser" name="申请人" type="string" variable="applyUserId" writable="false"></activiti:formProperty>
        <activiti:formProperty id="usage" name="用途" type="bigtext" expression="${usage}" writable="false"></activiti:formProperty>
        <activiti:formProperty id="amountMoney" name="总金额" type="double" writable="false"></activiti:formProperty>
        <activiti:formProperty id="treasurerApproved" name="是否同意" type="enum">
            <activiti:value id="true" name="同意"></activiti:value>
            <activiti:value id="false" name="驳回"></activiti:value>
        </activiti:formProperty>
    </extensionElements>
</userTask>

<sequenceFlow id="flow-treasurerAudit" name="财务不同意" sourceRef="exclusivegateway-treasurerAudit" targetRef="errorendevent2">
    <extensionElements>
        <activiti:executionListener event="take" class="com.sxdx.workflow.activiti.rest.listener.HandleErrorInfoForPaymentListener"></activiti:executionListener>
    </extensionElements>
    <conditionExpression xsi:type="tFormalExpression"><![CDATA[${treasurerApproved == 'false'}]]></conditionExpression>
</sequenceFlow>

<endEvent id="errorendevent2" name="End">
     <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
</endEvent>
```

可以看到，财务驳回时执行了一个任务监听器，并且流程流转到**异常结束事件**，抛出了一个异常代码：`PAYMENT_REJECT`,任务监听器代码实现如下：

```java
package com.sxdx.workflow.activiti.rest.listener;

import org.activiti.engine.delegate.DelegateExecution;
import org.activiti.engine.delegate.ExecutionListener;
import org.springframework.stereotype.Component;

@Component
public class HandleErrorInfoForPaymentListener implements ExecutionListener {
    @Override
    public void notify(DelegateExecution delegateExecution) {
        String activitiId = delegateExecution.getCurrentActivityId();
        if ("flow-treasurerAudit".equals(activitiId)){
            delegateExecution.setVariable("message","财务审批不通过");
        }else if ("flow-generalManagerAudit".equals(activitiId)){
            delegateExecution.setVariable("message","总经理审批不通过");
        }
    }
}
```

### 1.2.1 设置当前操作人

```yaml
# 项目相关配置
workflow:
  # 当前操作人
  operator: treasurer
```

1.2.1 获取待办

![image-20200715152507895](/images/activiti6-13/image-20200715152507895.png)

### 1.2.2 签收任务

![image-20200715152605388](/images/activiti6-13/image-20200715152605388.png)

### 1.2.3  获取任务表单

![image-20200715152741570](/images/activiti6-13/image-20200715152741570.png)

```json
{
  "code": "0000",
  "data": {
    "treasurerApproved": {
      "true": "同意",
      "false": "驳回"
    },
    "taskFormData": {
      "formKey": null,
      "deploymentId": "65001",
      "formProperties": [
        {
          "id": "applyUser",
          "name": "申请人",
          "type": {
            "name": "string",
            "mimeType": "text/plain"
          },
          "value": "kafeitu",
          "readable": true,
          "writable": false,
          "required": false
        },
        {
          "id": "usage",
          "name": "用途",
          "type": {
            "name": "bigtext",
            "mimeType": "text/plain"
          },
          "value": "计算机1000台",
          "readable": true,
          "writable": false,
          "required": false
        },
        {
          "id": "amountMoney",
          "name": "总金额",
          "type": {
            "name": "double"
          },
          "value": "100000.0",
          "readable": true,
          "writable": false,
          "required": false
        },
        {
          "id": "treasurerApproved",
          "name": "是否同意",
          "type": {
            "name": "enum"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": false
        }
      ],
      "task": null
    }
  },
  "message": "读取Task的表单成功"
}
```



### 1.2.4 驳回任务

![image-20200715154030673](/images/activiti6-13/image-20200715154030673.png)

![image-20200715154213154](/images/activiti6-13/image-20200715154213154.png)

可以看到，抛出的异常已经被事件子流程捕获并通过。

## 1.3  捕获付费子流程异常

```xml
<subProcess id="catchErrorForPayment" name="捕获付费子流程异常" triggeredByEvent="true">
    <startEvent id="errorstartevent1" name="Error start">
        <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
    </startEvent>
    <serviceTask id="recordErrorInfo" name="记录异常信息" activiti:expression="${execution.setVariable('ERROR_INFO', message)}"></serviceTask>
    <endEvent id="endevent6" name="End"></endEvent>
    <sequenceFlow id="flow29" sourceRef="recordErrorInfo" targetRef="endevent6"></sequenceFlow>
    <sequenceFlow id="flow34" sourceRef="errorstartevent1" targetRef="recordErrorInfo"></sequenceFlow>
</subProcess>
```

我们来分析下执行子流程。

triggeredByEvent="true" ：表示这是一个执行子流程，没有这个就是子流程。

定义了一个异常启动事件，只要检测到有节点抛出了异常代码：`PAYMENT_REJECT`,就会自动发起。

然后，设置了一个Java Service任务，记录了异常信息。

## 1.4  Java Service任务

![image-20200715155238237](/images/activiti6-13/image-20200715155238237.png)

![image-20200715155259528](/images/activiti6-13/image-20200715155259528.png)



事件子流程介绍完毕