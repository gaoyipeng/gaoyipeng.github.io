---
title: Activiti（32） 替换Activiti用户和用户组
date: 2020-10-16 16:53:23
tags: [Activiti]
categories: [Activiti]
description: Activiti（32） 替换Activiti用户和用户组
typora-root-url: ..
password: kiki
---

介绍Activiti用户和用户组时我们使用的是activiti自带的用户表，这节我们使用RBAC的用户表代替activiti用户表。

## 1、创建视图

前面我们已经介绍过，此处使用视图来替代原先的用户表

### 1.1 表对应关系

| 系统表                         | activiti表        |
| ------------------------------ | ----------------- |
| base_user(用户表)              | act_id_user       |
| base_role（角色表）            | act_id_group      |
| base_user_role(用户角色对照表) | act_id_membership |

*用户组其实可以看做是我们平常说的角色*。

### 1.2 视图创建SQL

```sql
# 根据 base_user、base_role 和 base_user_role 创建视图
# 需要根据具体的 用户表 和 角色表 进行相应字段的调整

-- 删除自带的用户信息表    

DROP TABLE IF EXISTS ACT_ID_MEMBERSHIP;
DROP TABLE IF EXISTS ACT_ID_GROUP;
DROP TABLE IF EXISTS ACT_ID_USER;
 
-- 如果之前存在视图则删除
    
DROP VIEW  IF EXISTS ACT_ID_MEMBERSHIP;  
DROP VIEW  IF EXISTS ACT_ID_USER;  
DROP VIEW  IF EXISTS ACT_ID_GROUP;
 
CREATE OR REPLACE VIEW act_id_user  AS   
  SELECT  
    u.user_id    AS ID_,  
    1               AS REV_,  
    u.username     AS FIRST_,  
    ''              AS LAST_,  
    u.email         AS EMAIL_,  
    '000000'      AS PWD_,  
    ''              AS PICTURE_ID_  
  FROM base_user u;    

-- 角色视图
 
CREATE VIEW act_id_group   
AS  
  SELECT r.role_id AS ID_,
         r.rev_ AS REV_,
         r.role_name AS NAME_,
         r.type_ AS TYPE_
  FROM base_role r;  

-- 用户-角色对应视图

CREATE VIEW act_id_membership  
AS  
   SELECT (SELECT u.user_id FROM base_user u WHERE u.user_id=ur.user_id) AS USER_ID_,
       (SELECT r.role_id FROM base_role r WHERE r.role_id=ur.role_id) AS GROUP_ID_  
   FROM base_user_role ur;
```

![image-20201017223049528](/images/activiti6-32/image-20201017223049528.png)

可以看到表中只保留act_id_info，另外3张表使用视图代替。

### 1.3、禁止自动生成用户表

为了禁止activiti自动生成用户表，我们需要在application.yml中添加如下配置：`db-identity-used: false`

```yaml
# Spring配置
spring:
  # activiti 模块
  # 解决启动报错：class path resource [processes/] cannot be resolved to URL because it does not exist
  activiti:
    check-process-definitions: false
    # 检测身份信息表是否存在
    db-identity-used: false
```



## 2、验证

我们修改一下动态表单章节使用的`leave-dynamic-from.bpmn`请假流程。

![image-20201018112505899](/images/activiti6-32/image-20201018112505899.png)

我们以`部门领导审批`节点为例,之前`部门领导审批`节点的设置是这样的：

![image-20201018113528117](/images/activiti6-32/image-20201018113528117.png)

现在我们使用视图代替Activiti用户表后，userId不再是之前的deptLeader，应该改为3

![image-20201018113600942](/images/activiti6-32/image-20201018113600942.png)

其他节点类似，修改后的流程定义文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.kafeitu.me/demo/activiti/leave">
  <process id="leave-dynamic-from" name="请假流程-动态表单" isExecutable="true">
    <documentation>请假流程演示-动态表单</documentation>
    <startEvent id="startevent1" name="Start" activiti:initiator="applyUserId">
      <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <activiti:formProperty id="reason" name="请假原因" type="string" required="true"></activiti:formProperty>
      </extensionElements>
    </startEvent>
    <userTask id="deptLeaderAudit" name="部门领导审批" activiti:candidateGroups="3">
      <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
        <activiti:formProperty id="reason" name="请假原因" type="string" writable="false"></activiti:formProperty>
        <activiti:formProperty id="deptLeaderPass" name="审批意见" type="enum" required="true">
          <activiti:value id="true" name="同意"></activiti:value>
          <activiti:value id="false" name="不同意"></activiti:value>
        </activiti:formProperty>
      </extensionElements>
    </userTask>
    <exclusiveGateway id="exclusivegateway5" name="Exclusive Gateway"></exclusiveGateway>
    <userTask id="modifyApply" name="调整申请" activiti:assignee="${applyUserId}">
      <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <activiti:formProperty id="reason" name="请假原因" type="string" required="true"></activiti:formProperty>
        <activiti:formProperty id="reApply" name="重新申请" type="enum" required="true">
          <activiti:value id="true" name="重新申请"></activiti:value>
          <activiti:value id="false" name="取消申请"></activiti:value>
        </activiti:formProperty>
        <modeler:initiator-can-complete xmlns:modeler="http://activiti.com/modeler"><![CDATA[false]]></modeler:initiator-can-complete>
      </extensionElements>
    </userTask>
    <userTask id="hrAudit" name="人事审批" activiti:candidateGroups="5">
      <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
        <activiti:formProperty id="reason" name="请假原因" type="string" writable="false"></activiti:formProperty>
        <activiti:formProperty id="hrPass" name="审批意见" type="enum" required="true">
          <activiti:value id="true" name="同意"></activiti:value>
          <activiti:value id="false" name="不同意"></activiti:value>
        </activiti:formProperty>
      </extensionElements>
    </userTask>
    <exclusiveGateway id="exclusivegateway6" name="Exclusive Gateway"></exclusiveGateway>
    <userTask id="reportBack" name="销假" activiti:assignee="${applyUserId}">
      <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" writable="false"></activiti:formProperty>
        <activiti:formProperty id="reason" name="请假原因" type="string" writable="false"></activiti:formProperty>
        <activiti:formProperty id="reportBackDate" name="销假日期" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
        <modeler:initiator-can-complete xmlns:modeler="http://activiti.com/modeler"><![CDATA[false]]></modeler:initiator-can-complete>
      </extensionElements>
    </userTask>
    <endEvent id="endevent1" name="End"></endEvent>
    <exclusiveGateway id="exclusivegateway7" name="Exclusive Gateway"></exclusiveGateway>
    <sequenceFlow id="flow2" sourceRef="startevent1" targetRef="deptLeaderAudit"></sequenceFlow>
    <sequenceFlow id="flow3" sourceRef="deptLeaderAudit" targetRef="exclusivegateway5"></sequenceFlow>
    <sequenceFlow id="flow4" name="不同意" sourceRef="exclusivegateway5" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderPass == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow5" name="同意" sourceRef="exclusivegateway5" targetRef="hrAudit">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderPass == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow6" sourceRef="hrAudit" targetRef="exclusivegateway6"></sequenceFlow>
    <sequenceFlow id="flow7" name="同意" sourceRef="exclusivegateway6" targetRef="reportBack">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${hrPass == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow8" sourceRef="reportBack" targetRef="endevent1"></sequenceFlow>
    <sequenceFlow id="flow9" name="不同意" sourceRef="exclusivegateway6" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${hrPass == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow10" name="重新申请" sourceRef="exclusivegateway7" targetRef="deptLeaderAudit">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow11" sourceRef="modifyApply" targetRef="exclusivegateway7"></sequenceFlow>
    <sequenceFlow id="flow12" name="结束流程" sourceRef="exclusivegateway7" targetRef="endevent1">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'false'}]]></conditionExpression>
    </sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_leave-dynamic-from">
    <bpmndi:BPMNPlane bpmnElement="leave-dynamic-from" id="BPMNPlane_leave-dynamic-from">
      <bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
        <omgdc:Bounds height="30.0" width="30.0" x="10.0" y="90.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="deptLeaderAudit" id="BPMNShape_deptLeaderAudit">
        <omgdc:Bounds height="55.0" width="105.0" x="90.0" y="80.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway5" id="BPMNShape_exclusivegateway5">
        <omgdc:Bounds height="40.0" width="40.0" x="250.0" y="87.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="modifyApply" id="BPMNShape_modifyApply">
        <omgdc:Bounds height="55.0" width="105.0" x="218.0" y="190.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="hrAudit" id="BPMNShape_hrAudit">
        <omgdc:Bounds height="55.0" width="105.0" x="358.0" y="80.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway6" id="BPMNShape_exclusivegateway6">
        <omgdc:Bounds height="40.0" width="40.0" x="495.0" y="87.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="reportBack" id="BPMNShape_reportBack">
        <omgdc:Bounds height="55.0" width="105.0" x="590.0" y="80.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
        <omgdc:Bounds height="28.0" width="28.0" x="625.0" y="283.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway7" id="BPMNShape_exclusivegateway7">
        <omgdc:Bounds height="40.0" width="40.0" x="250.0" y="280.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2">
        <omgdi:waypoint x="39.99660595085598" y="105.31907672235864"></omgdi:waypoint>
        <omgdi:waypoint x="90.0" y="106.38297872340425"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow3" id="BPMNEdge_flow3">
        <omgdi:waypoint x="195.0" y="107.29411764705883"></omgdi:waypoint>
        <omgdi:waypoint x="250.078125" y="107.078125"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow4" id="BPMNEdge_flow4">
        <omgdi:waypoint x="270.0900900900901" y="126.90990990990991"></omgdi:waypoint>
        <omgdi:waypoint x="270.3755656108597" y="190.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow5" id="BPMNEdge_flow5">
        <omgdi:waypoint x="289.92907801418437" y="107.0709219858156"></omgdi:waypoint>
        <omgdi:waypoint x="358.0" y="107.31316725978647"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow6" id="BPMNEdge_flow6">
        <omgdi:waypoint x="463.0" y="107.2488038277512"></omgdi:waypoint>
        <omgdi:waypoint x="495.0952380952381" y="107.0952380952381"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow7" id="BPMNEdge_flow7">
        <omgdi:waypoint x="534.921875" y="107.078125"></omgdi:waypoint>
        <omgdi:waypoint x="590.0" y="107.29411764705883"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow10" id="BPMNEdge_flow10">
        <omgdi:waypoint x="250.15503875968992" y="299.84496124031006"></omgdi:waypoint>
        <omgdi:waypoint x="142.0" y="299.0"></omgdi:waypoint>
        <omgdi:waypoint x="142.42819843342036" y="135.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow8" id="BPMNEdge_flow8">
        <omgdi:waypoint x="641.9920844327177" y="135.0"></omgdi:waypoint>
        <omgdi:waypoint x="639.25853110552" y="283.002387286845"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow11" id="BPMNEdge_flow11">
        <omgdi:waypoint x="270.3333333333333" y="245.0"></omgdi:waypoint>
        <omgdi:waypoint x="270.12048192771084" y="280.12048192771084"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow9" id="BPMNEdge_flow9">
        <omgdi:waypoint x="514.8198198198198" y="126.81981981981981"></omgdi:waypoint>
        <omgdi:waypoint x="514.0" y="217.0"></omgdi:waypoint>
        <omgdi:waypoint x="323.0" y="217.39219712525667"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow12" id="BPMNEdge_flow12">
        <omgdi:waypoint x="289.83870967741933" y="299.83870967741933"></omgdi:waypoint>
        <omgdi:waypoint x="625.0004626646179" y="297.1138173767104"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

然后我们修改当前操作人配置 ，4表示这个流程是部门领导发起的：

![image-20201018122045599](/images/activiti6-32/image-20201018122045599.png)

我们发起一个流程（注意添加请求头）：

![image-20201018122342465](/images/activiti6-32/image-20201018122342465.png)

![image-20201018122406474](/images/activiti6-32/image-20201018122406474.png)

然后部门经理获取待办，可以看到已经返回了待办信息：

![image-20201018122510979](/images/activiti6-32/image-20201018122510979.png)

## 3、Spring-Security获取当前操作人