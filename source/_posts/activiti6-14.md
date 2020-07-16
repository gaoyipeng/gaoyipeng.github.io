---
title: Activiti（14） 调用活动
date: 2020-07-15 16:04:01
tags: Activiti
categories: Activiti
description: Activiti（14） 调用活动
typora-root-url: ..
password: kiki
---

​	  **调用活动（Call Activiti）**就是用来解决流程业务中重复设计的问题，只需要设计一个通用流程供其他的流程调用即可。这样当这个公共的部分发生变化时，无需大范围改动，只修改这个公用部分即可。

​	   如果说子流程是一个类中的private方法，只能当前类调用。那么调用活动就是把一个通用方法提取到一个单独的类中供所有类使用。

​	与子流程不同的是：调用活动是以“引用”的方式把一个流程作为主流程的一个活动，当输出流到达调用活动时，流程引擎自动启动一个独立的流程。

# 1、调用活动

演示流程依然是办公用品采购，只是分为2部分，一个办公用品采购主流程，一个通用付费流程。在主流程付款节点，调用通用付款流程。

主流程：

![image-20200715165455427](/images/activiti6-14/image-20200715165455427.png)

通用付款流程：

![image-20200715170536367](/images/activiti6-14/image-20200715170536367.png)

## 1.1 流程定义

### 1.1.1 主流程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.kafeitu.me/activiti">
  <process id="purchase-callactivity" name="办公用品采购--callactivity方式" isExecutable="true">
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
    <userTask id="modifyApply" name="调整申请" activiti:assignee="${applyUserId}">
      <extensionElements>
        <activiti:formProperty id="dueDate" name="到货期限" type="date" expression="${dueDate}" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <activiti:formProperty id="listing" name="物品清单" type="bigtext" expression="${listing}" required="true"></activiti:formProperty>
        <activiti:formProperty id="amountMoney" name="总金额" type="double" expression="${amountMoney}" required="true"></activiti:formProperty>
      </extensionElements>
    </userTask>
    <exclusiveGateway id="exclusivegateway4" name="Exclusive Gateway"></exclusiveGateway>
    <sequenceFlow id="flow15" sourceRef="modifyApply" targetRef="exclusivegateway4"></sequenceFlow>
    <sequenceFlow id="flow16" name="调整后重新申请" sourceRef="exclusivegateway4" targetRef="deptLeaderAudit">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <endEvent id="endevent2" name="End"></endEvent>
    <sequenceFlow id="flow17" sourceRef="exclusivegateway4" targetRef="endevent2">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow19" sourceRef="exclusivegateway1" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderApproved == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow20" sourceRef="startevent1" targetRef="deptLeaderAudit"></sequenceFlow>
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
    <callActivity id="callactivity-payment" name="付款" calledElement="payment">
      <extensionElements>
        <activiti:in source="applyUserId" target="applyUserId"></activiti:in>
        <activiti:in source="listing" target="usage"></activiti:in>
        <activiti:in source="amountMoney" target="amountMoney"></activiti:in>
      </extensionElements>
    </callActivity>
    <sequenceFlow id="flow24" sourceRef="contactSupplier" targetRef="callactivity-payment"></sequenceFlow>
    <sequenceFlow id="flow29" sourceRef="callactivity-payment" targetRef="confirmReceipt"></sequenceFlow>
    <boundaryEvent id="boundaryerror1" name="Error" attachedToRef="callactivity-payment">
      <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
    </boundaryEvent>
    <sequenceFlow id="flow30" sourceRef="boundaryerror1" targetRef="modifyApply"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_purchase-callactivity">
    <bpmndi:BPMNPlane bpmnElement="purchase-callactivity" id="BPMNPlane_purchase-callactivity">
  	//.......
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

### 1.1.2 通用付款流程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.kafeitu.me/activiti">
  <process id="payment" name="通用付款流程" isExecutable="true">
    <startEvent id="startevent1" name="Start" activiti:initiator="applyUserId"></startEvent>
    <exclusiveGateway id="exclusivegateway2" name="Exclusive Gateway"></exclusiveGateway>
    <sequenceFlow id="flow5" sourceRef="treasurerAudit" targetRef="exclusivegateway5"></sequenceFlow>
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
    <exclusiveGateway id="exclusivegateway3" name="Exclusive Gateway"></exclusiveGateway>
    <sequenceFlow id="flow7" sourceRef="generalManagerAudit" targetRef="exclusivegateway3"></sequenceFlow>
    <userTask id="pay" name="出纳付款" activiti:candidateGroups="cashier">
      <extensionElements>
        <activiti:formProperty id="applyUser" name="申请人" type="string" variable="applyUserId" writable="false"></activiti:formProperty>
        <activiti:formProperty id="usage" name="用途" type="bigtext" expression="${usage}" writable="false"></activiti:formProperty>
        <activiti:formProperty id="supplier" name="受款方" type="string" writable="false"></activiti:formProperty>
        <activiti:formProperty id="bankName" name="开户行" type="string" writable="false"></activiti:formProperty>
        <activiti:formProperty id="bankAccount" name="银行账号" type="string" writable="false"></activiti:formProperty>
      </extensionElements>
    </userTask>
    <sequenceFlow id="flow8" sourceRef="exclusivegateway3" targetRef="pay">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${generalManagerApproved == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <endEvent id="endevent1" name="End"></endEvent>
    <sequenceFlow id="flow9" sourceRef="pay" targetRef="endevent1"></sequenceFlow>
    <sequenceFlow id="flow10" name="总经理不同意" sourceRef="exclusivegateway3" targetRef="errorendevent1">
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
    <sequenceFlow id="flow25" sourceRef="startevent1" targetRef="treasurerAudit"></sequenceFlow>
    <exclusiveGateway id="exclusivegateway5" name="Exclusive Gateway"></exclusiveGateway>
    <sequenceFlow id="flow26" sourceRef="exclusivegateway5" targetRef="exclusivegateway2">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${treasurerApproved == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow27" name="财务不同意" sourceRef="exclusivegateway5" targetRef="errorendevent2">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${treasurerApproved == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <endEvent id="errorendevent2" name="End">
      <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
    </endEvent>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_payment">
    <bpmndi:BPMNPlane bpmnElement="payment" id="BPMNPlane_payment">
      //..............
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

## 1.2 流程部署

这2个流程是独立的，此处需要部署2个流程。

![image-20200715181440830](/images/activiti6-14/image-20200715181440830.png)

## 1.3  发起流程

和子流程是一样的。

![image-20200715182144527](/images/activiti6-14/image-20200715182144527.png)

领导审批节点和联系供货方和前面操作步骤一样。我们略过，直接进入付款节点。

![image-20200715183657168](/images/activiti6-14/image-20200715183657168.png)



## 1.4 付款（调用付款流程）

```xml
<callActivity id="callactivity-payment" name="付款" calledElement="payment">
      <extensionElements>
        <activiti:in source="applyUserId" target="applyUserId"></activiti:in>
        <activiti:in source="listing" target="usage"></activiti:in>
        <activiti:in source="amountMoney" target="amountMoney"></activiti:in>
      </extensionElements>
</callActivity>

<boundaryEvent id="boundaryerror1" name="Error" attachedToRef="callactivity-payment">
    <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
</boundaryEvent>
```

`<callActivity>`是调用活动的意思，可以看到设置了calledElement="payment"，payment是付款通用流程的ID。这样当到达付款节点后会自动发起一个通用付款流程。我们看一下数据库，可以看到生成了一条流程实例。

![image-20200715191950279](/images/activiti6-14/image-20200715191950279.png)

我们查看一下流程图

![image-20200715192137148](/images/activiti6-14/image-20200715192137148.png)

已经发起了一个通用付款流程。然后接下来的操作就和子流程一样了。我们不再做演示。

# 2、数据分析

我们来看一下本条流程产生了哪些数据。

流程实例：2条，因为这2个流程是独立的，所以会产生2条记录

![image-20200715194643293](/images/activiti6-14/image-20200715194643293.png)

执行实例：5条（3条主流程产生的数据，2条子流程产生的数据）

![image-20200716090723458](/images/activiti6-14/image-20200716090723458.png)

标蓝的3条是主流程产生的数据，而在前两章子流程或执行子流程调用付款流程时只会生成2条数据，调用活动会多出的一条记录（85011），是流程运行至“付费”调用活动时产生的。

而最下面的2条数据，就是通用付款产生的执行实例。

前面2章我们已经了解到子流程可以共享主流程的变量，但调用活动除了会创建一个附属于子流程的流程实例外，子流程和主流程的变量是分离开来的。下图可以证明这个：

![image-20200716100512029](/images/activiti6-14/image-20200716100512029.png)

可以看到，2个流程的`execution_id`有2个。

通用付款流程的3个变量，都是主流程中的`callActiviti`通过`activiti:in`来传入的。



OK，调用活动就介绍到这了，实际使用还需要结合公司业务情况。