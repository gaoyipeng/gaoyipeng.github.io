---
title: Activiti（11） 多实例（会签）
date: 2020-07-02 16:51:27
tags: Activiti
categories: Activiti
description: Activiti（11） 多实例（会签）
typora-root-url: ..
password: kiki
---

# 1、场景引入

现在我们有这样的一个需求，在一个任务节点的时候，我们需要多个人对这个节点进行审批，比如实际中这样一个例子，有5个人，那么当5个人都投票的时候大概分为如下几种：

1. 所有人都去投票，当所有人都投票完成的时候，这个节点结束，流程运转到下一个节点。(并行)
2. 所有人都去投票,只要有任意2/3的人同意，这个节点结束，流程运转到下一个节点。(部分人投票只要满足条件就算完成)。
3. 5人中有一个部门经理，只要部门经理投票过了，这个节点结束，流程运转到下一个节点（一票否决权）。
4. 部门中根据职位不同，不同的人都不同的权重，当满足条件的时候，这个节点结束，流程运转到下一个节点。比如说a有0.2的权重，其他的四个人分别是0.1的权重，我们可以配置权重达到0.3就可以走向下一个节点，换言之a的权重是其他人的2倍，那就是a的投票相当于2个人投票。
5. 部门所有人都去投票，a投票结束到b,b开始投票结束到c,一直如此，串行执行。最终到最后一个人再统计结果，决定流程的运转。（串行）

上面的五种情况，我们可以提取出来一些信息，我们的activiti 工作流引擎，必须支持如下功能，才能满足上面的需求：

1. 任务节点可以配置自定义满足条件。
2. 任务节点必须支持串行、并行。
3. 任务节点必须支持可以指定候选人或者候选组。
4. 任务节点必须支持可以循环的次数。
5. 任务节点必须支持可以自定义权重。
6. 任务节点必须支持加签、减签。(就是动态的修改任务节点的处理人)

因为实际上的需求可能比上面的几种情况更加的复杂，上面的6个满足条件，工作流支持前4个，后面的2个条件是不支持的。

# 2、并行会签

我们依旧使用请假流程来演示，此处我们合并部门领导和人事节点，合并为一个【部门/人事联合会签】。

## 2.1  流程定义

![image-20200702174847574](/images/activiti6-11/image-20200702174847574.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC"
	xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath"
	targetNamespace="http://www.activiti.org/test">
	<process id="leave-countersign" name="请假流程-会签" isExecutable="true">
		<documentation>请假流程演示-会签</documentation>
		<startEvent id="startevent1" name="Start" activiti:initiator="applyUserId">
			<extensionElements>
				<activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
				<activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
				<activiti:formProperty id="reason" name="请假原因" type="string" required="true"></activiti:formProperty>
				<activiti:formProperty id="users" name="审批参与人" type="users" required="true"></activiti:formProperty>
				<activiti:formProperty id="validScript" type="javascript" default="alert('表单已经加载完毕');"></activiti:formProperty>
			</extensionElements>
		</startEvent>
		<userTask id="countersign" name="[部门/人事]联合会签" activiti:assignee="${user}">
			<extensionElements>
				<activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
				<activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
				<activiti:formProperty id="reason" name="请假原因" type="string" writable="false"></activiti:formProperty>
				<activiti:formProperty id="approved" name="审批意见" type="enum" required="true">
					<activiti:value id="true" name="同意"></activiti:value>
					<activiti:value id="false" name="拒绝"></activiti:value>
				</activiti:formProperty>
				<activiti:taskListener event="complete" delegateExpression="${leaveCounterSignCompleteListener}"></activiti:taskListener>
			</extensionElements>
			<multiInstanceLoopCharacteristics isSequential="false" activiti:collection="users"
				activiti:elementVariable="user"></multiInstanceLoopCharacteristics>
		</userTask>
		<userTask id="reportBack" name="销假" activiti:assignee="${applyUserId}">
			<extensionElements>
				<activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
				<activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
				<activiti:formProperty id="reason" name="请假原因" type="string" writable="false"></activiti:formProperty>
				<activiti:formProperty id="reportBackDate" name="销假日期" type="date" default="${endDate}" datePattern="yyyy-MM-dd"
					required="true"></activiti:formProperty>
			</extensionElements>
		</userTask>
		<endEvent id="endevent1" name="End"></endEvent>
		<sequenceFlow id="flow2" sourceRef="startevent1" targetRef="countersign">
			<extensionElements>
				<activiti:executionListener event="take" expression="${execution.setVariable('approvedCounter', 0)}"></activiti:executionListener>
			</extensionElements>
		</sequenceFlow>
		<sequenceFlow id="flow8" name="销假" sourceRef="reportBack" targetRef="endevent1">
			<extensionElements>
				<activiti:executionListener event="take" expression="${execution.setVariable('result', 'ok')}"></activiti:executionListener>
			</extensionElements>
		</sequenceFlow>
		<sequenceFlow id="flow13" name="全部通过" sourceRef="exclusivegateway1" targetRef="reportBack">
			<conditionExpression xsi:type="tFormalExpression"><![CDATA[${approvedCounter == users.size()}]]></conditionExpression>
		</sequenceFlow>
		<exclusiveGateway id="exclusivegateway1" name="Exclusive Gateway"></exclusiveGateway>
		<sequenceFlow id="flow14" sourceRef="countersign" targetRef="exclusivegateway1"></sequenceFlow>
		<userTask id="modifyApply" name="调整申请" activiti:assignee="${applyUserId}">
			<extensionElements>
				<activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
				<activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
				<activiti:formProperty id="reason" name="请假原因" type="string" required="true"></activiti:formProperty>
				<activiti:formProperty id="reApply" name="重新申请" type="enum" required="true">
					<activiti:value id="true" name="重新申请"></activiti:value>
					<activiti:value id="false" name="取消申请"></activiti:value>
				</activiti:formProperty>
			</extensionElements>
		</userTask>
		<sequenceFlow id="flow15" name="部分通过" sourceRef="exclusivegateway1" targetRef="modifyApply">
			<conditionExpression xsi:type="tFormalExpression"><![CDATA[${approvedCounter < users.size()}]]></conditionExpression>
		</sequenceFlow>
		<exclusiveGateway id="exclusivegateway2" name="Exclusive Gateway"></exclusiveGateway>
		<sequenceFlow id="flow16" sourceRef="modifyApply" targetRef="exclusivegateway2"></sequenceFlow>
		<sequenceFlow id="flow17" sourceRef="exclusivegateway2" targetRef="endevent1">
			<conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'false'}]]></conditionExpression>
		</sequenceFlow>
		<sequenceFlow id="flow18" name="重新申请" sourceRef="exclusivegateway2" targetRef="countersign">
			<extensionElements>
				<activiti:executionListener event="take" expression="${execution.setVariable('approvedCounter', 0)}"></activiti:executionListener>
			</extensionElements>
			<conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'true'}]]></conditionExpression>
		</sequenceFlow>
	</process>
	<bpmndi:BPMNDiagram id="BPMNDiagram_leave-countersign">
		<bpmndi:BPMNPlane bpmnElement="leave-countersign" id="BPMNPlane_leave-countersign">
			<bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
				<omgdc:Bounds height="35.0" width="35.0" x="10.0" y="30.0"></omgdc:Bounds>
			</bpmndi:BPMNShape>
			<bpmndi:BPMNShape bpmnElement="reportBack" id="BPMNShape_reportBack">
				<omgdc:Bounds height="55.0" width="105.0" x="340.0" y="20.0"></omgdc:Bounds>
			</bpmndi:BPMNShape>
			<bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
				<omgdc:Bounds height="35.0" width="35.0" x="375.0" y="203.0"></omgdc:Bounds>
			</bpmndi:BPMNShape>
			<bpmndi:BPMNShape bpmnElement="countersign" id="BPMNShape_countersign">
				<omgdc:Bounds height="55.0" width="105.0" x="90.0" y="20.0"></omgdc:Bounds>
			</bpmndi:BPMNShape>
			<bpmndi:BPMNShape bpmnElement="exclusivegateway1" id="BPMNShape_exclusivegateway1">
				<omgdc:Bounds height="40.0" width="40.0" x="220.0" y="27.0"></omgdc:Bounds>
			</bpmndi:BPMNShape>
			<bpmndi:BPMNShape bpmnElement="modifyApply" id="BPMNShape_modifyApply">
				<omgdc:Bounds height="55.0" width="105.0" x="188.0" y="110.0"></omgdc:Bounds>
			</bpmndi:BPMNShape>
			<bpmndi:BPMNShape bpmnElement="exclusivegateway2" id="BPMNShape_exclusivegateway2">
				<omgdc:Bounds height="40.0" width="40.0" x="220.0" y="200.0"></omgdc:Bounds>
			</bpmndi:BPMNShape>
			<bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2">
				<omgdi:waypoint x="45.0" y="47.0"></omgdi:waypoint>
				<omgdi:waypoint x="90.0" y="47.0"></omgdi:waypoint>
			</bpmndi:BPMNEdge>
			<bpmndi:BPMNEdge bpmnElement="flow8" id="BPMNEdge_flow8">
				<omgdi:waypoint x="392.0" y="75.0"></omgdi:waypoint>
				<omgdi:waypoint x="392.0" y="203.0"></omgdi:waypoint>
				<bpmndi:BPMNLabel>
					<omgdc:Bounds height="11.0" width="100.0" x="-22.0" y="-17.0"></omgdc:Bounds>
				</bpmndi:BPMNLabel>
			</bpmndi:BPMNEdge>
			<bpmndi:BPMNEdge bpmnElement="flow13" id="BPMNEdge_flow13">
				<omgdi:waypoint x="260.0" y="47.0"></omgdi:waypoint>
				<omgdi:waypoint x="340.0" y="47.0"></omgdi:waypoint>
				<bpmndi:BPMNLabel>
					<omgdc:Bounds height="11.0" width="100.0" x="-29.0" y="-17.0"></omgdc:Bounds>
				</bpmndi:BPMNLabel>
			</bpmndi:BPMNEdge>
			<bpmndi:BPMNEdge bpmnElement="flow14" id="BPMNEdge_flow14">
				<omgdi:waypoint x="195.0" y="47.0"></omgdi:waypoint>
				<omgdi:waypoint x="220.0" y="47.0"></omgdi:waypoint>
			</bpmndi:BPMNEdge>
			<bpmndi:BPMNEdge bpmnElement="flow15" id="BPMNEdge_flow15">
				<omgdi:waypoint x="240.0" y="67.0"></omgdi:waypoint>
				<omgdi:waypoint x="240.0" y="110.0"></omgdi:waypoint>
				<bpmndi:BPMNLabel>
					<omgdc:Bounds height="11.0" width="100.0" x="10.0" y="0.0"></omgdc:Bounds>
				</bpmndi:BPMNLabel>
			</bpmndi:BPMNEdge>
			<bpmndi:BPMNEdge bpmnElement="flow16" id="BPMNEdge_flow16">
				<omgdi:waypoint x="240.0" y="165.0"></omgdi:waypoint>
				<omgdi:waypoint x="240.0" y="200.0"></omgdi:waypoint>
			</bpmndi:BPMNEdge>
			<bpmndi:BPMNEdge bpmnElement="flow17" id="BPMNEdge_flow17">
				<omgdi:waypoint x="260.0" y="220.0"></omgdi:waypoint>
				<omgdi:waypoint x="375.0" y="220.0"></omgdi:waypoint>
			</bpmndi:BPMNEdge>
			<bpmndi:BPMNEdge bpmnElement="flow18" id="BPMNEdge_flow18">
				<omgdi:waypoint x="220.0" y="220.0"></omgdi:waypoint>
				<omgdi:waypoint x="142.0" y="220.0"></omgdi:waypoint>
				<omgdi:waypoint x="142.0" y="75.0"></omgdi:waypoint>
				<bpmndi:BPMNLabel>
					<omgdc:Bounds height="11.0" width="100.0" x="9.0" y="14.0"></omgdc:Bounds>
				</bpmndi:BPMNLabel>
			</bpmndi:BPMNEdge>
		</bpmndi:BPMNPlane>
	</bpmndi:BPMNDiagram>
</definitions>
```

其中 isSequential 

- true：表示串行，即节点人员顺序审批
- false：表示并行，即节点人员同步审批

```xml
<multiInstanceLoopCharacteristics isSequential="false" activiti:collection="users"
				activiti:elementVariable="user"></multiInstanceLoopCharacteristics>
```

## 2.2 部署流程

![image-20200703105612630](/images/activiti6-11/image-20200703105612630.png)

## 2.3 发起流程

我们先来查看下启动节点的内容：

```xml
<startEvent id="startevent1" name="Start" activiti:initiator="applyUserId">
    <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <activiti:formProperty id="reason" name="请假原因" type="string" required="true"></activiti:formProperty>
        <activiti:formProperty id="users" name="审批参与人" type="users" required="true"></activiti:formProperty>
        <activiti:formProperty id="validScript" type="javascript" default="alert('表单已经加载完毕');"></activiti:formProperty>
    </extensionElements>
</startEvent>
```

type="users"以及type="javascript"，不是流程引擎支持的类型。需要我们自定义类型。

### 2.3.1 自定义类型

前面我们学习动态表单时介绍过Activiti支持的表单字段类型。

![image-20200705162151989](/images/activiti6-11/image-20200705162151989.png)

现在我们学习如何自定义字段类型。

1. 创建解析类，需要继承`AbstractFormType`类

   ```java
   package com.sxdx.workflow.activiti.rest.entity.formTypes;
   
   import cn.hutool.core.util.ObjectUtil;
   import freemarker.template.utility.StringUtil;
   import org.activiti.engine.form.AbstractFormType;
   import org.springframework.stereotype.Component;
   
   import java.util.Arrays;
   
   @Component
   public class KikiUserFormType extends AbstractFormType {
       /**
        * 把表单中的值转换为实际的对象
        * @param s
        * @return
        */
       @Override
       public Object convertFormValueToModelValue(String s) {
           String[] split = StringUtil.split(s, ',');
           return Arrays.asList(split);
       }
   
       /**
        * 把实际对象的值转换为表单中的值
        * @param o
        * @return
        */
       @Override
       public String convertModelValueToFormValue(Object o) {
           return ObjectUtil.toString(0);
       }
   
       /**
        * 定义表单类型的标识符
        */
       @Override
       public String getName() {
           return "users";
       }
   }
   
   ```

   ```java
   package com.sxdx.workflow.activiti.rest.entity.formTypes;
   
   import org.activiti.engine.form.AbstractFormType;
   import org.springframework.stereotype.Component;
   
   @Component
   public class JavascriptFormType extends AbstractFormType {
       /**
        * 把表单中的值转换为实际的对象
        * @param propertyValue
        * @return
        */
       @Override
       public Object convertFormValueToModelValue(String propertyValue) {
           return propertyValue;
       }
   
       /**
        * 把实际对象的值转换为表单中的值
        * @param modelValue
        * @return
        */
       @Override
       public String convertModelValueToFormValue(Object modelValue) {
           return (String) modelValue;
       }
   
       /**
        * 定义表单类型的标识符
        *
        * @return
        */
       @Override
       public String getName() {
           return "javascript";
       }
   }
   
   ```

   

2. 在流程引擎中注册解析类

   此处用到了BeanPostProcessor（后置Bean处理器），对这个不熟悉的可以参考我的另一篇博客 ：[Spring源码学习（7）：Bean的生命周期](https://blog.gaoyp.cn/2020/04/12/spring-study-07/)

   ```java
   /**
        * 自定义表单字段类型
        * @return
        */
       @Bean
       public BeanPostProcessor activitiConfigurer() {
           return new BeanPostProcessor() {
               @Override
               public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
                   if (bean instanceof SpringProcessEngineConfiguration) {
                       List<AbstractFormType> customFormTypes = Arrays.<AbstractFormType>asList(javascriptFormType,kikiUserFormType);
                       ((SpringProcessEngineConfiguration)bean).setCustomFormTypes(customFormTypes);
                   }
                   return bean;
               }
               @Override
               public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                   return bean;
               }
           };
   
       }
   ```

   

## 2.4 获取流程启动节点表单信息

![image-20200705171609395](/images/activiti6-11/image-20200705171609395.png)

返回信息：(可以看到我们的自定义类型users,javascript)

```json
{
  "code": "0000",
  "data": {
    "form": {
      "formKey": null,
      "deploymentId": "2501",
      "formProperties": [
        {
          "id": "startDate",
          "name": "请假开始日期",
          "type": {
            "name": "date"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": true
        },
        {
          "id": "endDate",
          "name": "请假结束日期",
          "type": {
            "name": "date"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": true
        },
        {
          "id": "reason",
          "name": "请假原因",
          "type": {
            "name": "string",
            "mimeType": "text/plain"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": true
        },
        {
          "id": "users",
          "name": "审批参与人",
          "type": {
            "name": "users"
          },
          "value": "0",
          "readable": true,
          "writable": true,
          "required": true
        },
        {
          "id": "validScript",
          "name": null,
          "type": {
            "name": "javascript"
          },
          "value": "alert('表单已经加载完毕');",
          "readable": true,
          "writable": true,
          "required": false
        }
      ],
      "processDefinition": null
    }
  },
  "message": "获取启动节点"
}
```

## 2.5 发起流程







参考：https://blog.csdn.net/qq_30739519/article/details/51239818

https://stackoverflow.com/questions/28022772/how-to-declare-an-activiti-custom-formtype-in-a-spring-boot-application