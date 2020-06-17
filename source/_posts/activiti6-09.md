---
title: Activiti（8） 外置表单实战
date: 2020-06-16 15:55:02
tags: Activiti
categories: Activiti
description: Activiti（8） 外置表单实战
typora-root-url: ..
password: kiki
---

> 这节我们继续学习外置表单。这种方式常用于基于工作流平台开发的方式，代码写的很少，开发人员只要把表单内容写好保存到**.form**文件中即可，然后配置每个节点需要的表单名称（form key），实际运行时通过引擎提供的API读取Task对应的form内容输出到页面。
>
> 此种方式对于在经常添加新流程的需求比较适用，可以快速发布新流程，把流程设计出来之后再设计表单之后两者关联就可以使用了。

动态表单有一个弊端，就是每个节点的表单信息需要我们动态的构建。而且每个表单包含的字段不尽相同，这样就需要我们的开发人员针对每个节点的表单信息生成对应的页面。这样是不合理的。我们来看看相同流程情况下，外置表单如何实现。

# 1、流程定义

**leave-formkey.bpmn：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.kafeitu.me/activiti">
  <process id="leave-formkey" name="请假流程-外置表单" isExecutable="true">
    <documentation>外置表单</documentation>
    <startEvent id="startevent1" name="Start" activiti:initiator="applyUserId" activiti:formKey="leave-formkey/leave-start.form"></startEvent>
    <userTask id="deptLeaderVerify" name="部门经理审批" activiti:candidateGroups="deptLeader" activiti:formKey="leave-formkey/approve-deptLeader.form"></userTask>
    <exclusiveGateway id="exclusivegateway1" name="Exclusive Gateway"></exclusiveGateway>
    <userTask id="hrVerify" name="人事经理审批" activiti:candidateGroups="hr" activiti:formKey="leave-formkey/approve-hr.form"></userTask>
    <exclusiveGateway id="exclusivegateway2" name="Exclusive Gateway"></exclusiveGateway>
    <userTask id="reportBack" name="销假" activiti:assignee="${applyUserId}" activiti:formKey="leave-formkey/report-back.form"></userTask>
    <endEvent id="endevent1" name="End"></endEvent>
    <userTask id="modifyApply" name="调整申请内容" activiti:assignee="${applyUserId}" activiti:formKey="leave-formkey/modify-apply.form"></userTask>
    <exclusiveGateway id="exclusivegateway3" name="Exclusive Gateway"></exclusiveGateway>
    <sequenceFlow id="flow1" sourceRef="startevent1" targetRef="deptLeaderVerify"></sequenceFlow>
    <sequenceFlow id="flow2" sourceRef="deptLeaderVerify" targetRef="exclusivegateway1"></sequenceFlow>
    <sequenceFlow id="flow3" name="同意" sourceRef="exclusivegateway1" targetRef="hrVerify">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderApproved == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow4" sourceRef="hrVerify" targetRef="exclusivegateway2"></sequenceFlow>
    <sequenceFlow id="flow5" name="同意" sourceRef="exclusivegateway2" targetRef="reportBack">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${hrApproved == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow6" sourceRef="reportBack" targetRef="endevent1">
      <extensionElements>
        <activiti:executionListener event="take" expression="${execution.setVariable('result', 'ok')}"></activiti:executionListener>
      </extensionElements>
    </sequenceFlow>
    <sequenceFlow id="flow7" name="不同意" sourceRef="exclusivegateway2" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${hrApproved == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow8" name="不同意" sourceRef="exclusivegateway1" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderApproved == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow9" sourceRef="modifyApply" targetRef="exclusivegateway3"></sequenceFlow>
    <sequenceFlow id="flow10" name="调整后继续申请" sourceRef="exclusivegateway3" targetRef="deptLeaderVerify">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow11" name="取消申请，并设置取消标志" sourceRef="exclusivegateway3" targetRef="endevent1">
      <extensionElements>
        <activiti:executionListener event="take" expression="${execution.setVariable('result', 'canceled')}"></activiti:executionListener>
      </extensionElements>
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <textAnnotation id="textannotation1" textFormat="text/plain">
      <text>请求被驳回后员工可以选择继续申请，或者取消本次申请</text>
    </textAnnotation>
    <association id="association1" sourceRef="modifyApply" targetRef="textannotation1"></association>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_leave-formkey">
    <bpmndi:BPMNPlane bpmnElement="leave-formkey" id="BPMNPlane_leave-formkey">
      <bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="10.0" y="50.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="deptLeaderVerify" id="BPMNShape_deptLeaderVerify">
        <omgdc:Bounds height="55.0" width="105.0" x="90.0" y="40.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway1" id="BPMNShape_exclusivegateway1">
        <omgdc:Bounds height="40.0" width="40.0" x="230.0" y="47.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="hrVerify" id="BPMNShape_hrVerify">
        <omgdc:Bounds height="55.0" width="105.0" x="310.0" y="40.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway2" id="BPMNShape_exclusivegateway2">
        <omgdc:Bounds height="40.0" width="40.0" x="470.0" y="47.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="reportBack" id="BPMNShape_reportBack">
        <omgdc:Bounds height="55.0" width="105.0" x="580.0" y="40.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="615.0" y="213.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="modifyApply" id="BPMNShape_modifyApply">
        <omgdc:Bounds height="55.0" width="105.0" x="198.0" y="120.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway3" id="BPMNShape_exclusivegateway3">
        <omgdc:Bounds height="40.0" width="40.0" x="230.0" y="210.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="textannotation1" id="BPMNShape_textannotation1">
        <omgdc:Bounds height="57.0" width="112.0" x="340.0" y="168.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="flow1" id="BPMNEdge_flow1">
        <omgdi:waypoint x="45.0" y="67.0"></omgdi:waypoint>
        <omgdi:waypoint x="90.0" y="67.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2">
        <omgdi:waypoint x="195.0" y="67.0"></omgdi:waypoint>
        <omgdi:waypoint x="230.0" y="67.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow3" id="BPMNEdge_flow3">
        <omgdi:waypoint x="270.0" y="67.0"></omgdi:waypoint>
        <omgdi:waypoint x="310.0" y="67.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="22.0" x="269.0" y="50.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow4" id="BPMNEdge_flow4">
        <omgdi:waypoint x="415.0" y="67.0"></omgdi:waypoint>
        <omgdi:waypoint x="470.0" y="67.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow5" id="BPMNEdge_flow5">
        <omgdi:waypoint x="510.0" y="67.0"></omgdi:waypoint>
        <omgdi:waypoint x="580.0" y="67.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="22.0" x="529.0" y="50.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow6" id="BPMNEdge_flow6">
        <omgdi:waypoint x="632.0" y="95.0"></omgdi:waypoint>
        <omgdi:waypoint x="632.0" y="213.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow7" id="BPMNEdge_flow7">
        <omgdi:waypoint x="490.0" y="87.0"></omgdi:waypoint>
        <omgdi:waypoint x="490.0" y="147.0"></omgdi:waypoint>
        <omgdi:waypoint x="303.0" y="147.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="33.0" x="438.0" y="119.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow8" id="BPMNEdge_flow8">
        <omgdi:waypoint x="250.0" y="87.0"></omgdi:waypoint>
        <omgdi:waypoint x="250.0" y="120.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="33.0" x="260.0" y="87.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow9" id="BPMNEdge_flow9">
        <omgdi:waypoint x="250.0" y="175.0"></omgdi:waypoint>
        <omgdi:waypoint x="250.0" y="210.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow10" id="BPMNEdge_flow10">
        <omgdi:waypoint x="230.0" y="230.0"></omgdi:waypoint>
        <omgdi:waypoint x="142.0" y="230.0"></omgdi:waypoint>
        <omgdi:waypoint x="142.0" y="95.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="77.0" x="159.0" y="210.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow11" id="BPMNEdge_flow11">
        <omgdi:waypoint x="270.0" y="230.0"></omgdi:waypoint>
        <omgdi:waypoint x="615.0" y="230.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="33.0" width="100.0" x="58.0" y="-37.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="association1" id="BPMNEdge_association1">
        <omgdi:waypoint x="303.0" y="147.0"></omgdi:waypoint>
        <omgdi:waypoint x="396.0" y="168.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

可以看到没有了`<activiti:formProperty/>` ，换成 `activiti:formKey=`  设置外置表单，我们以start节点的表单为例：

```html
<div class="control-group">
	<label class="control-label" for="startDate">开始时间：</label>
	<div class="controls">
		<input type="text" id="startDate" name="startDate" class="datepicker" data-date-format="yyyy-mm-dd" required />
	</div>
</div>
<div class="control-group">
	<label class="control-label" for="endDate">结束时间：</label>
	<div class="controls">
		<input type="text" id="endDate" name="endDate" class="datepicker" data-date-format="yyyy-mm-dd" required />
	</div>
</div>
<div class="control-group">
	<label class="control-label" for="reason">请假原因：</label>
	<div class="controls">
		<textarea id="reason" name="reason" required></textarea>
	</div>
</div>
```



## 1.1 初始化流程审批人员

审批人员和上一篇文章一样，不做修改

## 1.2  部署流程定义

把流程定义文件和节点表单一起打包，然后部署。

![image-20200617145244947](/images/activiti6-09/image-20200617145244947.png)

![image-20200617145316486](/images/activiti6-09/image-20200617145316486.png)

部署成功后可以看到数据库中对应的记录。

![image-20200617150312307](/images/activiti6-09/image-20200617150312307.png)

![image-20200617145354040](/images/activiti6-09/image-20200617145354040.png)

## 1.3  获取流程启动节点表单信息

我们针对外置表单，新建一个`FormKeyController.java`文件，外置表单专有的方法写这里：

```java
package com.sxdx.workflow.activiti.rest.controller;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import lombok.extern.slf4j.Slf4j;
import org.activiti.engine.FormService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@Api(value="外置表单模块", description="外置表单模块")
@RestController
@Slf4j
@RequestMapping("/formKey")
public class FormKeyController {
    @Autowired
    private FormService formService;

    @GetMapping(value = "get-form/start/{processDefinitionId}")
    @ApiOperation(value = "获取外置表单-开始表单的内容",notes = "获取外置表单-开始表单的内容")
    public Object findStartForm(@PathVariable("processDefinitionId") @ApiParam("流程定义id") String processDefinitionId) throws Exception {
        // 根据流程定义ID读取外置表单
        Object startForm = formService.getRenderedStartForm(processDefinitionId);
        return startForm;
    }

}

```

下面是测试结果，可以看到返回了HTML片段，前端只需把这些返回片段直接显示在页面即可。

![image-20200617171440325](/images/activiti6-09/image-20200617171440325.png)

## 1.4  发起流程

**这里我们优化一下**，外置表单发起流程的接口是和动态表单一样的，为了后期好维护。这里我们把DynamicFromController`类中`/start-process/{processDefinitionId}`请求对应的代码迁移到`ProcessController`中。

```java
/**
     * 提交启动流程
     */
    @PostMapping(value = "/start-process/{processDefinitionId}")
    @ApiOperation(value = "提交启动流程",notes = "提交启动流程，key需要以fp_开头")
    public CommonResponse submitStartFormAndStartProcessInstance(@PathVariable("processDefinitionId") String processDefinitionId,
                                                                 HttpServletRequest request) throws CommonException {
        ProcessInstance processInstance = processService.submitStartFormAndStartProcessInstance(processDefinitionId,request);
        return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("流程启动成功，流程ID：" + processInstance.getId()).data(processInstance.getId());
    }

 /**
     * 获取动态表单提交的数据，并发起流程，表单数据 key-value结构，key需要以fp_开头
     * @param processDefinitionId
     * @param request
     * @return
     */
    @Override
    @Transactional
    public ProcessInstance submitStartFormAndStartProcessInstance(String processDefinitionId,HttpServletRequest request) throws CommonException {
        Map<String, String> formProperties = new HashMap<String, String>();
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
        }else {
            throw new CommonException("表单数据为空");
        }

        log.debug("start form parameters: {}", formProperties);

        //TODO 此处应该添加获取当前操作人的代码,先写死用kafeitu发起流程。用户未登录不能操作，实际应用使用权限框架实现，例如Spring Security、Shiro等
        UserQueryImpl user = new UserQueryImpl();
        user = (UserQueryImpl)identityService.createUserQuery().userId("kafeitu");

        ProcessInstance processInstance = null;
        try {
            identityService.setAuthenticatedUserId(user.getId());
            processInstance = formService.submitStartFormData(processDefinitionId, formProperties);
            log.debug("start a processinstance: {}", processInstance);
        } finally {
            identityService.setAuthenticatedUserId(null);
        }
        return processInstance;
    }
```

测试：

![image-20200617183656129](/images/activiti6-09/image-20200617183656129.png)