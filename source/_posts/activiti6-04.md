---
title: Activiti（4）七大Service接口
date: 2020-05-29 09:09:47
tags: Activiti
categories: Activiti
description: Activiti（4）七大Service接口
typora-root-url: ..
password: kiki
---

> 本节我们介绍Activiti提供的 7大Service接口。先了解这7个service的用途，有利于我们更好的学习activiti.

![image-20200529100807061](/images/activiti/activiti6-04/image-20200529100807061.png)

# 1、流程模型

我们通过使用下面这个请假流程模型来练习这7个Activiti Service。

## 1.1 流程图

流程节点的具体内容，可以使用Eclipse Designer来查看。

![image-20200529101645437](/images/activiti/activiti6-04/image-20200529101645437.png)

以下是各个节点需要注意的点，gif 图片将就着看吧~~~

![leave.gif](/images/activiti/activiti6-04/leave.gif)

## 1.2 流程定义

以下是上面流程图对应的定义文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.kafeitu.me/activiti/leave">
  <process id="leave" name="请假流程-普通表单" isExecutable="true">
    <documentation>请假流程演示</documentation>
    <startEvent id="startevent1" name="Start" activiti:initiator="applyUserId"></startEvent>
    <userTask id="deptLeaderVerify" name="部门领导审批" activiti:candidateGroups="deptLeader"></userTask>
    <exclusiveGateway id="exclusivegateway5" name="Exclusive Gateway"></exclusiveGateway>
    <userTask id="modifyApply" name="调整申请" activiti:assignee="${applyUserId}"></userTask>
    <userTask id="hrVerify" name="人事审批" activiti:candidateGroups="hr"></userTask>
    <exclusiveGateway id="exclusivegateway6" name="Exclusive Gateway"></exclusiveGateway>
    <userTask id="reportBack" name="销假" activiti:assignee="${applyUserId}">
      <extensionElements>
        <activiti:taskListener event="complete" delegateExpression="${reportBackEndProcessor}"></activiti:taskListener>
      </extensionElements>
    </userTask>
    <endEvent id="endevent1" name="End"></endEvent>
    <exclusiveGateway id="exclusivegateway7" name="Exclusive Gateway"></exclusiveGateway>
    <sequenceFlow id="flow2" sourceRef="startevent1" targetRef="deptLeaderVerify"></sequenceFlow>
    <sequenceFlow id="flow3" sourceRef="deptLeaderVerify" targetRef="exclusivegateway5"></sequenceFlow>
    <sequenceFlow id="flow4" name="不同意" sourceRef="exclusivegateway5" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${!deptLeaderApproved}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow5" name="同意" sourceRef="exclusivegateway5" targetRef="hrVerify">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderApproved}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow6" sourceRef="hrVerify" targetRef="exclusivegateway6"></sequenceFlow>
    <sequenceFlow id="flow7" name="同意" sourceRef="exclusivegateway6" targetRef="reportBack">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${hrApproved}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow8" sourceRef="reportBack" targetRef="endevent1"></sequenceFlow>
    <sequenceFlow id="flow9" name="不同意" sourceRef="exclusivegateway6" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${!hrApproved}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow10" name="重新申请" sourceRef="exclusivegateway7" targetRef="deptLeaderVerify">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow11" sourceRef="modifyApply" targetRef="exclusivegateway7"></sequenceFlow>
    <sequenceFlow id="flow12" name="结束流程" sourceRef="exclusivegateway7" targetRef="endevent1">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${!reApply}]]></conditionExpression>
    </sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_leave">
    <bpmndi:BPMNPlane bpmnElement="leave" id="BPMNPlane_leave">
      <bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="0.0" y="46.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="deptLeaderVerify" id="BPMNShape_deptLeaderVerify">
        <omgdc:Bounds height="55.0" width="105.0" x="80.0" y="36.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway5" id="BPMNShape_exclusivegateway5">
        <omgdc:Bounds height="40.0" width="40.0" x="240.0" y="43.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="modifyApply" id="BPMNShape_modifyApply">
        <omgdc:Bounds height="55.0" width="105.0" x="208.0" y="114.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="hrVerify" id="BPMNShape_hrVerify">
        <omgdc:Bounds height="55.0" width="105.0" x="348.0" y="36.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway6" id="BPMNShape_exclusivegateway6">
        <omgdc:Bounds height="40.0" width="40.0" x="485.0" y="43.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="reportBack" id="BPMNShape_reportBack">
        <omgdc:Bounds height="55.0" width="105.0" x="580.0" y="36.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="615.0" y="195.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway7" id="BPMNShape_exclusivegateway7">
        <omgdc:Bounds height="40.0" width="40.0" x="240.0" y="192.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2">
        <omgdi:waypoint x="35.0" y="63.0"></omgdi:waypoint>
        <omgdi:waypoint x="80.0" y="63.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow3" id="BPMNEdge_flow3">
        <omgdi:waypoint x="185.0" y="63.0"></omgdi:waypoint>
        <omgdi:waypoint x="240.0" y="63.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow4" id="BPMNEdge_flow4">
        <omgdi:waypoint x="260.0" y="83.0"></omgdi:waypoint>
        <omgdi:waypoint x="260.0" y="114.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="33.0" x="270.0" y="83.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow5" id="BPMNEdge_flow5">
        <omgdi:waypoint x="280.0" y="63.0"></omgdi:waypoint>
        <omgdi:waypoint x="348.0" y="63.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="22.0" x="300.0" y="46.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow6" id="BPMNEdge_flow6">
        <omgdi:waypoint x="453.0" y="63.0"></omgdi:waypoint>
        <omgdi:waypoint x="485.0" y="63.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow7" id="BPMNEdge_flow7">
        <omgdi:waypoint x="525.0" y="63.0"></omgdi:waypoint>
        <omgdi:waypoint x="580.0" y="63.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="22.0" x="539.0" y="46.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow8" id="BPMNEdge_flow8">
        <omgdi:waypoint x="632.0" y="91.0"></omgdi:waypoint>
        <omgdi:waypoint x="632.0" y="195.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow9" id="BPMNEdge_flow9">
        <omgdi:waypoint x="505.0" y="83.0"></omgdi:waypoint>
        <omgdi:waypoint x="504.0" y="141.0"></omgdi:waypoint>
        <omgdi:waypoint x="313.0" y="141.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="33.0" x="515.0" y="83.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow10" id="BPMNEdge_flow10">
        <omgdi:waypoint x="240.0" y="212.0"></omgdi:waypoint>
        <omgdi:waypoint x="132.0" y="212.0"></omgdi:waypoint>
        <omgdi:waypoint x="132.0" y="91.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="44.0" x="142.0" y="192.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow11" id="BPMNEdge_flow11">
        <omgdi:waypoint x="260.0" y="169.0"></omgdi:waypoint>
        <omgdi:waypoint x="260.0" y="192.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow12" id="BPMNEdge_flow12">
        <omgdi:waypoint x="280.0" y="212.0"></omgdi:waypoint>
        <omgdi:waypoint x="615.0" y="212.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="44.0" x="429.0" y="219.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

1.3 节点审批人员约定

| 用户名           | ACT_ID_USER(表) | ACT_ID_GROUP(表) |
| ---------------- | --------------- | ---------------- |
| 发起人(startmen) |                 |                  |
| 部门领导         | deptmen         | deptLeader       |
| 人事领导         | hrmen           | hr               |

# 2、单元测试

## 2.1 获取流程引擎及各个Service接口

> 只有先获取了流程引擎，才能获取其他的Service接口。

```java
package com.sxdx.workflow.activiti.rest;

import org.activiti.engine.*;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import static org.junit.Assert.assertNotNull;

@SpringBootTest
@RunWith(SpringRunner.class)
public class ActivitiServiceTest {

    private ProcessEngine processEngine;
    private IdentityService identityService;
    private RepositoryService repositoryService;
    private RuntimeService runtimeService;
    private TaskService taskService;
    private HistoryService historyService;


    @Test
    public void before(){
        ProcessEngineConfiguration processEngineConfiguration = ProcessEngineConfiguration
                .createStandaloneProcessEngineConfiguration();
        processEngineConfiguration.setJdbcDriver("com.mysql.cj.jdbc.Driver");
        processEngineConfiguration.setJdbcUrl("jdbc:mysql://localhost:3306/activiti-demo?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true");
        processEngineConfiguration.setJdbcUsername("root");
        processEngineConfiguration.setJdbcPassword("xxxxxxx");
        //如果表不存在，则自动创建表
  processEngineConfiguration.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
        processEngine = processEngineConfiguration.buildProcessEngine();
        System.out.println(processEngine.toString());
        
        repositoryService = processEngine.getRepositoryService();
        identityService = processEngine.getIdentityService();
        runtimeService = processEngine.getRuntimeService();
        taskService = processEngine.getTaskService();
        historyService = processEngine.getHistoryService();
        
        assertNotNull(processEngine);
    }

}

```

测试结果：查看 activiti-demo 数据库，可以看到生成了28张表。

![image-20200529164306561](/images/activiti/activiti6-04/image-20200529164306561.png)

## 2.2 初始化审批人

```java
package com.sxdx.workflow.activiti.rest;

import org.activiti.engine.*;
import org.activiti.engine.identity.User;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNotNull;

@SpringBootTest
@RunWith(SpringRunner.class)
public class ActivitiServiceTest {

    private ProcessEngine processEngine;
    private IdentityService identityService;
    private RepositoryService repositoryService;
    private RuntimeService runtimeService;
    private TaskService taskService;
    private HistoryService historyService;

    /**
     * 获取流程引擎及各个Service
     */
    @Before
    public void before(){
        ProcessEngineConfiguration processEngineConfiguration = ProcessEngineConfiguration
                .createStandaloneProcessEngineConfiguration();
        processEngineConfiguration.setJdbcDriver("com.mysql.cj.jdbc.Driver");
        processEngineConfiguration.setJdbcUrl("jdbc:mysql://localhost:3306/activiti-demo?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true");
        processEngineConfiguration.setJdbcUsername("root");
        processEngineConfiguration.setJdbcPassword("gaoyipeng");
       //如果表不存在，则自动创建表
       processEngineConfiguration.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
        processEngine = processEngineConfiguration.buildProcessEngine();
        System.out.println(processEngine.toString());

        repositoryService = processEngine.getRepositoryService();
        identityService = processEngine.getIdentityService();
        runtimeService = processEngine.getRuntimeService();
        taskService = processEngine.getTaskService();
        historyService = processEngine.getHistoryService();

        assertNotNull(processEngine);
    }


    /**
     * 初始化审批人 act_id_user: deptmen， hrmen
     */
    @Test
    public void initUser(){
        User deptmen = identityService.newUser("deptmen");
        deptmen.setFirstName("部门领导");
        identityService.saveUser(deptmen);

        User hrmen = identityService.newUser("hrmen");
        hrmen.setFirstName("人事领导");
        identityService.saveUser(hrmen);

        assertEquals(2,identityService.createUserQuery().count());
    }
}

```

执行结果：

![image-20200529143545904](/images/activiti/activiti6-04/image-20200529143545904.png)



## 2.3 初始化组

```java
    /**
     * 初始化组 act_id_group: deptLeader, hr
     * 在Activiti中组分为2种：
     *   assignment：普通的岗位角色，是用户分配业务中的功能权限
     *   security-role: 安全角色，全局管理用户组织及整个流程的状态
     *   如果使用Activiti提供的Explorer，需要security-role才能看到manage页签，需要assignment才能claim任务
     */
    @Test
    public void initGroup(){
        Group deptLeader = identityService.newGroup("deptLeader");
        deptLeader.setName("deptLeader");
        //扩展字段
        deptLeader.setType("assignment");
        identityService.saveGroup(deptLeader);

        Group hr = identityService.newGroup("hr");
        hr.setName("hr");
        hr.setType("assignment");
        identityService.saveGroup(hr);

        assertEquals(2,identityService.createGroupQuery().count());
    } 
```

执行结果：

![image-20200529145814813](/images/activiti/activiti6-04/image-20200529145814813.png)



## 2.4 初始化人员、组的关系

```java
    /**
     * 初始化人员、组的关系
     */
    @Test
    public void initMemberShip(){
        identityService.createMembership("deptmen","deptLeader");
        identityService.createMembership("hrmen","hr");
    }
```

执行结果：

![image-20200529150116825](/images/activiti/activiti6-04/image-20200529150116825.png)



API 补充：

```java
//删除user
identityService.deleteUser(userId);
//删除group
identityService.deleteGroup(groupId);
//删除user、group关联关系
identityService.deleteMembership( userId,  groupId);
```



## 2.5 部署流程定义

将下载好的leave.bpmn放到`src/main/resources`目录下

```java
    /**
     * 部署流程定义
     */
    @Test
    public void deployTest(){
        Deployment deployment = repositoryService.createDeployment()
                .addClasspathResource("leave.bpmn")
                .deploy();
        assertNotNull(deployment);
    }
```

执行结果：

![image-20200529152439777](/images/activiti/activiti6-04/image-20200529152439777.png)

![image-20200529152450279](/images/activiti/activiti6-04/image-20200529152450279.png)

## 2.6 发起审批

```java
    /**
     * 发起流程
     */
    @Test
    public void submitApplyTest(){
        String applyUserId = "startmen";
        //设置流程启动发起人,在流程开始之前设置，会自动在表ACT_HI_PROCINST 中的START_USER_ID_中设置用户ID
        identityService.setAuthenticatedUserId(applyUserId);
        runtimeService.startProcessInstanceByKey("leave");
    }
```

执行结果：

![image-20200529154857621](/images/activiti/activiti6-04/image-20200529154857621.png)

![image-20200529154905575](/images/activiti/activiti6-04/image-20200529154905575.png)

## 2.7 部门领导获取待办

```java
 /**
     * 获取待办
     */
    @Test
    public void getTaskTodo(){
        //根据当前人id查询待办任务
        List<Task> taskList = taskService.createTaskQuery()
                .processDefinitionKey("leave")
                .taskAssignee("deptmen")
                .active().list();
        //根据当前人未签收的任务
        List<Task> taskList1 = taskService.createTaskQuery()
                .processDefinitionKey("leave")
                .taskCandidateUser("deptmen")
                .active().list();

        List<Task> list = new ArrayList<>();
        list.addAll(taskList);
        list.addAll(taskList1);
        System.out.println("-------"+list.get(0).toString()+"----"+list.get(0).getName());
        assertEquals(1,list.size());

        Task task = list.get(0);
    }
```

断点截图：

![image-20200529155701816](/images/activiti/activiti6-04/image-20200529155701816.png)

## 2.8 部门领导审批通过流程

```java
/**
     * 获取待办并通过
     */
    @Test
    public void getTaskTodo(){
        //根据当前人id查询待办任务
        List<Task> taskList = taskService.createTaskQuery()
                .processDefinitionKey("leave")
                .taskAssignee("deptmen")
                .active().list();
        //根据当前人未签收的任务
        List<Task> taskList1 = taskService.createTaskQuery()
                .processDefinitionKey("leave")
                .taskCandidateUser("deptmen")
                .active().list();

        List<Task> list = new ArrayList<>();
        list.addAll(taskList);
        list.addAll(taskList1);
        System.out.println("-------"+list.get(0).toString()+"----"+list.get(0).getName());
        assertEquals(1,list.size());

        Task task = list.get(0);

        //审批流程
        ProcessInstance processInstance = runtimeService.createProcessInstanceQuery()
                .processDefinitionKey("leave")
                .singleResult();
        // 添加批注
        identityService.setAuthenticatedUserId("deptmen");
        taskService.addComment(task.getId(), processInstance.getId(), "deptmen【同意】了");
        Map<String, Object> variables = new HashMap<>();
        variables.put("deptLeaderApproved", true);
        // 只有签收任务，act_hi_taskinst 表的 assignee 字段才不为 null
        taskService.claim(task.getId(), "deptmen");
        taskService.complete(task.getId(), variables);
        
    }
```

执行结果：可以看到审批意见了

![image-20200529160354305](/images/activiti/activiti6-04/image-20200529160354305.png)

## 2.9 获取已办

```java
    /**
     * 获取已完成的流程
     */
    @Test
    public void getCompileTask(){
        List<HistoricTaskInstance> taskInstanceList = historyService.createHistoricTaskInstanceQuery()
                .processDefinitionKey("leave")
                .taskAssignee("deptmen")
                .finished().list();
        for (HistoricTaskInstance instance  :taskInstanceList) {
            System.out.println(instance.getProcessInstanceId()+"--"+instance.getName()+"--"+instance.getAssignee());
        }
    }
```

执行结果：可以看到 5001 这条流程，部门经理已经审批过了。

![image-20200529160953048](/images/activiti/activiti6-04/image-20200529160953048.png)

## 2.10  获取审批意见

```java
/**
     * 获取审批意见
     */
    @Test
    public void getHistoryComment(){
        //获取流程实例对象
        ProcessInstance processInstance = runtimeService.createProcessInstanceQuery()
                .processDefinitionKey("leave").singleResult();
        //获取历史活动集合
        List<HistoricActivityInstance> historicActivityInstanceList = historyService.createHistoricActivityInstanceQuery()
                .processInstanceId(processInstance.getId())
                .activityType("userTask")
                .finished()
                .list();

        for (HistoricActivityInstance historicActivityInstance:historicActivityInstanceList ) {
            List<Comment> commentList = taskService.getTaskComments(historicActivityInstance.getTaskId(), "comment");
            for (int i = 0; i < commentList.size(); i++) {
                System.out.println(commentList.get(i).getProcessInstanceId()+"---"+ commentList.get(i).getUserId() +"批复内容：" + commentList.get(i).getFullMessage());
            }
        }
    }
```

![image-20200529161311961](/images/activiti/activiti6-04/image-20200529161311961.png)

**请记住这个5001，获取流程图时会用到。**

常用的Service已经介绍完毕，有需要再添加其他用例。

# 3、获取流程图

这个是比较固定的代码，不做详解介绍。只做演示。添加代码如下

![image-20200601092940565](/images/activiti/activiti6-04/image-20200601092940565.png)

访问API：http://localhost:8080/process/read-resource/5001  ，pProcessInstanceId是流程实例：5001（**act_hi_procinst**表ID_）

```java
 @RequestMapping(value = "/read-resource/{pProcessInstanceId}")
    public void readResource(@PathVariable("pProcessInstanceId")String pProcessInstanceId, HttpServletResponse response)
            throws Exception {
        // 设置页面不缓存
        response.setHeader("Pragma", "No-cache");
        response.setHeader("Cache-Control", "no-cache");
        response.setDateHeader("Expires", 0);

        String processDefinitionId = "";
        ProcessInstance processInstance = runtimeService.createProcessInstanceQuery().processInstanceId(pProcessInstanceId).singleResult();
        if(processInstance == null) {
            HistoricProcessInstance historicProcessInstance = historyService.createHistoricProcessInstanceQuery().processInstanceId(pProcessInstanceId).singleResult();
            processDefinitionId = historicProcessInstance.getProcessDefinitionId();
        } else {
            processDefinitionId = processInstance.getProcessDefinitionId();
        }
        ProcessDefinitionQuery pdq = repositoryService.createProcessDefinitionQuery();
        ProcessDefinition pd = pdq.processDefinitionId(processDefinitionId).singleResult();

        String resourceName = pd.getDiagramResourceName();

        if(resourceName.endsWith(".png") && StringUtils.isEmpty(pProcessInstanceId) == false)
        {
            getActivitiProccessImage(pProcessInstanceId,response);
            //ProcessDiagramGenerator.generateDiagram(pde, "png", getRuntimeService().getActiveActivityIds(processInstanceId));
        }
        else
        {
            // 通过接口读取
            InputStream resourceAsStream = repositoryService.getResourceAsStream(pd.getDeploymentId(), resourceName);

            // 输出资源内容到相应对象
            byte[] b = new byte[1024];
            int len = -1;
            while ((len = resourceAsStream.read(b, 0, 1024)) != -1) {
                response.getOutputStream().write(b, 0, len);
            }
        }
    }
```

获取的流程图如下：

![image-20200601093323407](/images/activiti/activiti6-04/image-20200601093323407.png)



本节到此结束。