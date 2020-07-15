---
title: Activiti（12） 子流程
date: 2020-07-10 09:02:12
tags: Activiti
categories: Activiti
description: Activiti（12） 子流程
typora-root-url: ..
password: kiki
---

# 1、基本概念

​		**子流程**和**调用活动**的设计思想就是为了使流程设计更合理，更方便。在代码中，一个方法过多时会拆分为多个方法。流程设计也是如此。

​		把不同阶段的流程任务抽取出来作为一个**子流程（Subprocess）**，这样当业务发生变化时只需要聚焦在不同的子流程即可，主流程负责串联多个子流程。

​		除了子流程，还有一个**事件子流程**，它和子流程类似，把一系列的活动归结到一起处理，不同的是事件子流程不能直接启动，而要“被动”的由其它事件触发启动。

​		此时产生了一个问题，如果这 个子流程功能很通用，在多个流程中都用到了，难道我们每个流程都定义一次吗？如果后期子流程业务改动，需要修改所有流程中的子流程，这样非常不利于后期的修改维护。

​	  **调用活动（Call Activiti）**就是用来解决流程业务中重复设计的问题，只需要设计一个通用流程供其他的流程调用即可。这样当这个公共的部分发生变化时，无需大范围改动，只修改这个公用部分即可。

​	如果说子流程是一个类中的private方法，只能当前类调用。那么调用活动就是把一个通用方法提取到一个单独的类中供所有类使用。

# 2、子流程

我们使用《Activiti实战》中的办公用品采购流程来走一遍子流程。眼过千遍不如手过一遍。

![image-20200711175102100](/images/jvm/activiti6-12/image-20200711175102100.png)



## 2.1 流程定义

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.kafeitu.me/activiti">
  <process id="purchase-subprocess" name="办公用品采购" isExecutable="true">
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
      <sequenceFlow id="flow25" sourceRef="startevent2" targetRef="treasurerAudit"></sequenceFlow>
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
    </subProcess>
    <boundaryEvent id="boundaryerror1" name="Error" attachedToRef="subprocessPay">
      <errorEventDefinition errorRef="PAYMENT_REJECT"></errorEventDefinition>
    </boundaryEvent>
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
    <sequenceFlow id="flow21" name="捕获子流程的异常事件" sourceRef="boundaryerror1" targetRef="modifyApply"></sequenceFlow>
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
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_purchase-subprocess">
    <bpmndi:BPMNPlane bpmnElement="purchase-subprocess" id="BPMNPlane_purchase-subprocess">
       //位置信息省略
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```





## 2.2 初始化人员

新添加角色：

| 角色名称 |    角色 ID     |
| :------: | :------------: |
| 财务人员 |   treasurer    |
|  出纳员  |    cashier     |
| 后勤人员 |  supportCrew   |
|  总经理  | generalManager |

新添加的用户：

|  用户名称   |  用户id   |     分配角色      |
| :---------: | :-------: | :---------------: |
| Tony Zhang  | treasurer | treasurer,cashier |
| Thomas Wang |  thomas   |    supportCrew    |
| Manager Li  |  manager  |  generalManager   |

我们直接使用代码来注入:

```java
/**
* 初始化审批人 act_id_user
*/
@Test
public void initUser(){
    User treasurer = identityService.newUser("treasurer");
    treasurer.setFirstName("Tony Zhang");
    identityService.saveUser(treasurer);

    User thomas = identityService.newUser("thomas");
    thomas.setFirstName("Thomas Wang");
    identityService.saveUser(thomas);
    
    User thomas = identityService.newUser("manager");
    thomas.setFirstName("Manager Li");
    identityService.saveUser(thomas);
}

/**
 * 初始化组 act_id_group
*/
@Test
public void initGroup(){
    Group treasurer = identityService.newGroup("treasurer");
    treasurer.setName("财务人员");
    treasurer.setType("assignment");
    identityService.saveGroup(treasurer);

    Group cashier = identityService.newGroup("cashier");
    cashier.setName("出纳员");
    cashier.setType("assignment");
    identityService.saveGroup(cashier);

    Group supportCrew = identityService.newGroup("supportCrew");
    supportCrew.setName("后勤人员");
    supportCrew.setType("assignment");
    identityService.saveGroup(supportCrew);
    
    Group generalManager = identityService.newGroup("generalManager");
    generalManager.setName("总经理");
    generalManager.setType("assignment");
    identityService.saveGroup(supportCrew);
}

/**
 * 初始化人员、组的关系
 */
@Test
public void initMemberShip(){
    identityService.createMembership("treasurer","treasurer");
    identityService.createMembership("treasurer","cashier");
    identityService.createMembership("thomas","supportCrew");
    identityService.createMembership("manager","generalManager");
}
```

## 2.3 部署流程定义

这个就不介绍了，前面已经演示过好几次了。

## 2.4 发起流程

启动节点：

```xml
<startEvent id="startevent1" name="startevent1" activiti:initiator="applyUserId">
      <extensionElements>
        <activiti:formProperty id="dueDate" name="到货期限" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <activiti:formProperty id="listing" name="物品清单" type="bigtext" required="true"></activiti:formProperty>
        <activiti:formProperty id="amountMoney" name="总金额" type="double" required="true"></activiti:formProperty>
      </extensionElements>
    </startEvent>
```

### 2.4.1  自定义表单类型

`bigtext`和`double`都不是表单默认类型，需要我们自定义，这个在上一节已经介绍过，我们直接贴出代码：

```java
package com.sxdx.workflow.activiti.rest.entity.formTypes;

import org.activiti.engine.impl.form.StringFormType;
import org.springframework.stereotype.Component;

/**
 * 大文本、即html的textarea，集成自引擎默认支持的string类型的实现
 */
@Component
public class BigtextFormType extends StringFormType {
    @Override
    public String getName() {
        return "bigtext";
    }
}
---------------------------------------------------------------------------------
 package com.sxdx.workflow.activiti.rest.entity.formTypes;

import cn.hutool.core.util.ObjectUtil;
import org.activiti.engine.form.AbstractFormType;
import org.springframework.stereotype.Component;

@Component
public class DoubleFormType extends AbstractFormType {
    @Override
    public Object convertFormValueToModelValue(String s) {
        return new Double(s);
    }

    @Override
    public String convertModelValueToFormValue(Object o) {
        return ObjectUtil.toString(o);
    }

    @Override
    public String getName() {
        return "double";
    }
}
----------------------------------------------------------------------------
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
                List<AbstractFormType> customFormTypes = Arrays.<AbstractFormType>asList(javascriptFormType,kikiUserFormType,bigtextFormType,doubleFormType);
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

### 2.4.2 获取启动节点表单信息

![image-20200713113501020](/images/activiti6-12/image-20200713113501020.png)

```json
{
  "code": "0000",
  "data": {
    "form": {
      "formKey": null,
      "deploymentId": "20005",
      "formProperties": [
        {
          "id": "dueDate",
          "name": "到货期限",
          "type": {
            "name": "date"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": true
        },
        {
          "id": "listing",
          "name": "物品清单",
          "type": {
            "name": "bigtext",
            "mimeType": "text/plain"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": true
        },
        {
          "id": "amountMoney",
          "name": "总金额",
          "type": {
            "name": "double"
          },
          "value": "null",
          "readable": true,
          "writable": true,
          "required": true
        }
      ],
      "processDefinition": null
    }
  },
  "message": "获取启动节点"
}
```



### 2.4.3 发起流程

使用`kafeitu`发起流程

![image-20200713111942450](/images/activiti6-12/image-20200713111942450.png)

## 2.5 部门领导审批

```xml
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
```

### 2.5.1 获取待办

![image-20200713122918528](/images/activiti6-12/image-20200713122918528.png)

### 2.5.2 签收任务

![image-20200713123043055](/images/activiti6-12/image-20200713123043055.png)

### 2.5.3 获取任务表单

![image-20200713140059325](/images/activiti6-12/image-20200713140059325.png)

### 2.5.4 完成任务

![image-20200713140358468](/images/activiti6-12/image-20200713140358468.png)

现阶段流程图：![image-20200713140600734](/images/activiti6-12/image-20200713140600734.png)

## 2.6  联系供货方

```xml
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
```

进入付款子流程,`subprocessPay`就是子流程的id。可以看到，后勤人员完成任务就，直接就进入子流程中了。

![image-20200713165937682](/images/activiti6-12/image-20200713165937682.png)

```xml
<sequenceFlow id="flow22" name="进入付费子流程" sourceRef="contactSupplier" targetRef="subprocessPay">
    <extensionElements>
        <activiti:executionListener event="take" expression="${execution.setVariable('usage', listing)}"></activiti:executionListener>
    </extensionElements>
</sequenceFlow>
```

### 2.6.1 设置当前操作人

设置`thomas`为当前操作人。前面我们都是通过硬编码的方式设置当前操作人。为了方便，我们把这个写成配置文件形式。

yaml文件添加配置

```yaml
# 项目相关配置
workflow:
  # 名称
  name: workflow-activiti-rest
  # 版本
  version: 1.0.0
  # 版权年份
  copyrightYear: 2020
  # 文件路径 示例（ Windows配置D:/ruoyi/uploadPath，Linux配置 /home/uploadPath）
  profile: D:/workflow/uploadPath
  # 当前操作人
  operator: thomas
```

```java
package com.sxdx.common.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * 全局配置类
 * 
 * @author ruoyi
 */
@Component
@ConfigurationProperties(prefix = "workflow")
public class GlobalConfig
{
   //省略其他代码
    private static String operator;


    public static String getOperator() {
        return operator;
    }

    public static void setOperator(String operator) {
        GlobalConfig.operator = operator;
    }
    
    public static String getOperator() {
        return operator;
    }

    public void setOperator(String operator) {
        GlobalConfig.operator = operator;
    }
}

```

下面这些地方都需要改

![image-20200713161444573](/images/activiti6-12/image-20200713161444573.png)

### 2.6.2  获取待办

![image-20200713163646892](/images/activiti6-12/image-20200713163646892.png)

### 2.6.3 签收任务

![image-20200713163732864](/images/activiti6-12/image-20200713163732864.png)

### 2.6.4  获取任务表单

![image-20200713165301891](/images/activiti6-12/image-20200713165301891.png)

```json
{
  "code": "0000",
  "data": {
    "taskFormData": {
      "formKey": null,
      "deploymentId": "20005",
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
          "id": "dueDate",
          "name": "到货期限",
          "type": {
            "name": "date"
          },
          "value": "2020-02-03",
          "readable": true,
          "writable": false,
          "required": false
        },
        {
          "id": "listing",
          "name": "物品清单",
          "type": {
            "name": "bigtext",
            "mimeType": "text/plain"
          },
          "value": "计算机1000000台",
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
          "id": "supplier",
          "name": "供货方",
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
          "id": "bankName",
          "name": "开户行",
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
          "id": "bankAccount",
          "name": "银行账号",
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
          "id": "planDate",
          "name": "预计交货日期",
          "type": {
            "name": "date"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": true
        }
      ],
      "task": null
    }
  },
  "message": "读取Task的表单成功"
}
```

### 2.6.5  完成任务

![image-20200713182502072](/images/activiti6-12/image-20200713182502072.png)

![image-20200714160539663](/images/activiti6-12/image-20200714160539663.png)

以上可以看到，已经顺利进入子流程中了。



## 2.7  财务审批

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
```

### 2.7.1 设置当前操作人

```yaml
# 项目相关配置
workflow:
  # 当前操作人
  operator: treasurer
```

### 2.7.2 获取待办

![image-20200714165446194](/images/activiti6-12/image-20200714165446194.png)

### 2.7.3  签收任务

![image-20200714165738486](/images/activiti6-12/image-20200714165738486.png)

### 2.7.4  获取任务表单

![image-20200714165812538](/images/activiti6-12/image-20200714165812538.png)

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
      "deploymentId": "20005",
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
          "value": "计算机1000000台",
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

可以看到`usage`字段也有了值，说明`进入付费子流程`的执行监听器成功执行了。

### 2.7.5 完成任务

![image-20200714170624219](/images/activiti6-12/image-20200714170624219.png)

实时流程图：

![image-20200714170704402](/images/activiti6-12/image-20200714170704402.png)

## 2.8 总经理审批

```xml
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
```

### 2.8.1 设置当前操作人

```yaml
# 项目相关配置
workflow:
  # 当前操作人
  operator: manager
```

### 2.8.2 获取待办

![image-20200714182634566](/images/activiti6-12/image-20200714182634566.png)

### 2.8.3 签收任务

![image-20200714183124821](/images/activiti6-12/image-20200714183124821.png)

### 2.8.4 获取任务表单

![image-20200714183153777](/images/activiti6-12/image-20200714183153777.png)

```json
{
  "code": "0000",
  "data": {
    "taskFormData": {
      "formKey": null,
      "deploymentId": "20005",
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
          "value": "计算机1000000台",
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
          "id": "generalManagerApproved",
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
    },
    "generalManagerApproved": {
      "true": "同意",
      "false": "驳回"
    }
  },
  "message": "读取Task的表单成功"
}
```



### 2.8.5 完成审批

![image-20200714183600557](/images/activiti6-12/image-20200714183600557.png)

## 2.9  出纳付款

```xml
 <userTask id="pay" name="出纳付款" activiti:candidateGroups="cashier">
     <extensionElements>
         <activiti:formProperty id="applyUser" name="申请人" type="string" variable="applyUserId" writable="false"></activiti:formProperty>
         <activiti:formProperty id="usage" name="用途" type="bigtext" expression="${usage}" writable="false"></activiti:formProperty>
         <activiti:formProperty id="supplier" name="受款方" type="string" writable="false"></activiti:formProperty>
         <activiti:formProperty id="bankName" name="开户行" type="string" writable="false"></activiti:formProperty>
         <activiti:formProperty id="bankAccount" name="银行账号" type="string" writable="false"></activiti:formProperty>
     </extensionElements>
</userTask>
```

审批流程和前面步骤是一样的。

![image-20200714184446203](/images/activiti6-12/image-20200714184446203.png)

## 2.10  收货确认

```xml
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
```

发起人收货确认即可，和前几步一样，最后结果：

![image-20200714184823599](/images/activiti6-12/image-20200714184823599.png)

# 3、数据分析

我们来看一下本条流程产生了哪些数据。

流程实例：1条
![image-20200714185652172](/images/activiti6-12/image-20200714185652172.png)

执行实例：2条

![image-20200714185851278](/images/activiti6-12/image-20200714185851278.png)

另外需要注意的就是“共享”变量。子流程可以获取流程发起人提供的`amountMoney`字段。这个字段是在子流程外赋值的，说明**子流程可以共享主流程所有变量**。

而且，子流程设置的变量也以主流程的执行实例ID为依据。

![image-20200715104229923](/images/activiti6-12/image-20200715104229923.png)

