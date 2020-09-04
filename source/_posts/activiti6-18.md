---
title: Activiti（18） 事务子流程
date: 2020-08-27 13:24:37
tags: Activiti
categories: Activiti
description: Activiti（18） 事务子流程
typora-root-url: ..
password: kiki
---

事务子流程是一种嵌入式子流程，可用于将多个活动组织在一个事务里。事务是工作的逻辑单元，可以组织一组独立活动，使得它们可以一起成功或失败。

**事务的可能结果：**事务有三种不同的结果：

- **事务成功**：若未被取消，或被意外终止，则事务*成功*。若事务子流程成功，将使用出口顺序流离开。若流程后面抛出了补偿事件，成功的事务可以被补偿。*请注意：*与“普通”嵌入式子流程一样，可以使用补偿抛出中间事件，在事务成功完成后补偿。
- **结束取消**：若执行到达取消结束事件时，事务被*取消*。当取消边界事件触发时，首先会中断当前范围的所有活动执行。接下来，启动事务范围内所有有效的的补偿边界事件（compensation boundary event）。补偿会同步执行，也就是说在离开事务前，边界事件会等待补偿完成。当补偿完成时，使用取消边界事件的出口顺序流，离开事务子流程。
- **错误结束**：若由于抛出了错误结束事件，且未被事务子流程所在的范围捕获，则事务会被*意外*终止（错误被事件子流程的边界捕获也一样）。在这种情况下，不会进行补偿。

说明：学习事务子流程需要先对**取消边界事件**和**取消结束事件**的概念有一定的了解。（可以参考《边界事件》和《Activiti结束事件》这2篇教程）

## 1、流程定义

此处我们画一个流程图来模拟事务可能的3种结果：

### 1.1 流程定义

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <process id="transactionSubProcess" isExecutable="true">
    <startEvent id="sid-F0F15ABD-37D9-4EAC-960B-9E099AFEBC6D"></startEvent>
    <endEvent id="sid-84E39C18-F341-4AE7-840E-213581EA1482"></endEvent>
    <sequenceFlow id="sid-588CFB0B-F353-4AD3-93B7-028E85CF20AF" sourceRef="sid-AC28574D-44AF-4650-BF00-9AB95E26B123" targetRef="sid-84E39C18-F341-4AE7-840E-213581EA1482"></sequenceFlow>
    <endEvent id="sid-614C6EFE-4E6D-4CD3-AB4D-E404CB716F13"></endEvent>
    <sequenceFlow id="sid-1190457B-AD79-42A1-AF82-D553997E2552" sourceRef="sid-C52F0179-A84E-4328-8996-69C01D3F372F" targetRef="sid-614C6EFE-4E6D-4CD3-AB4D-E404CB716F13"></sequenceFlow>
    <endEvent id="sid-DB9AC289-C0A7-4B15-A443-414784487BC6"></endEvent>
    <sequenceFlow id="sid-551B1C78-B4E1-4D45-9544-8D24117D7B91" sourceRef="sid-8A361F6E-ECF3-4EDF-AEB3-11E0C4FBC386" targetRef="sid-DB9AC289-C0A7-4B15-A443-414784487BC6"></sequenceFlow>
    <transaction id="sid-AC28574D-44AF-4650-BF00-9AB95E26B123" name="subProcess">
      <startEvent id="sid-46AEBDA5-3FEF-4EF8-8370-922D206FFC3D"></startEvent>
      <serviceTask id="sid-66402E45-A7AD-4C27-8CC3-156F0AA4CEE7" name="A打款" activiti:class="com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent.APaymentService"></serviceTask>
      <serviceTask id="sid-6E0604FC-8D7B-4F99-BE32-76EE9DE7905A" name="B收钱" activiti:class="com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent.BReceivePaymentService"></serviceTask>
      <exclusiveGateway id="sid-B0303DE4-1024-488D-82CD-78A4A1D2D336"></exclusiveGateway>
      <endEvent id="sid-6F8681D7-0158-460C-8929-DFB96AC5A217" name="错误结束">
        <errorEventDefinition></errorEventDefinition>
      </endEvent>
      <endEvent id="sid-57226B0E-783D-4FD4-87BB-C228F160AEAF" name="结束取消">
        <cancelEventDefinition></cancelEventDefinition>
      </endEvent>
      <serviceTask id="sid-A66E0771-CB55-4F3F-9D91-7FF0CE66FEE4" name="B收钱补偿" activiti:class="com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent.BMakeUpService" isForCompensation="true"></serviceTask>
      <serviceTask id="sid-18A68A51-A66F-4760-B610-A126D4EE9636" name="A打款补偿" activiti:class="com.sxdx.workflow.activiti.rest.serviceTask.boundaryCompensationEvent.AMakeUpService" isForCompensation="true"></serviceTask>
      <endEvent id="sid-D185BC86-F922-46B4-93C4-2D9C6559A57D" name="正常结束"></endEvent>
      <userTask id="sid-9F7131AC-5F2D-4B9B-ABE5-C1ED4DFCFA00" name="manage人工审批" activiti:candidateGroups="generalManager">
        <extensionElements>
          <activiti:formProperty id="pass" name="事务结果类型" type="string" required="true"></activiti:formProperty>
        </extensionElements>
      </userTask>
      <boundaryEvent id="sid-5F643838-FA37-41E9-8F1B-1FEDDAA15B45" attachedToRef="sid-66402E45-A7AD-4C27-8CC3-156F0AA4CEE7" cancelActivity="false">
        <compensateEventDefinition></compensateEventDefinition>
      </boundaryEvent>
      <boundaryEvent id="sid-5520B272-95AD-4301-AF1D-97FAF2A87E6F" attachedToRef="sid-6E0604FC-8D7B-4F99-BE32-76EE9DE7905A" cancelActivity="false">
        <compensateEventDefinition></compensateEventDefinition>
      </boundaryEvent>
      <sequenceFlow id="sid-3ED40954-C3F7-4360-BA68-BF1BD9FCDEB2" sourceRef="sid-46AEBDA5-3FEF-4EF8-8370-922D206FFC3D" targetRef="sid-66402E45-A7AD-4C27-8CC3-156F0AA4CEE7"></sequenceFlow>
      <sequenceFlow id="sid-1C0737B3-87B2-4C56-8A1F-D6EC2BB9A6C4" sourceRef="sid-66402E45-A7AD-4C27-8CC3-156F0AA4CEE7" targetRef="sid-6E0604FC-8D7B-4F99-BE32-76EE9DE7905A"></sequenceFlow>
      <sequenceFlow id="sid-EADDE25C-040A-4639-880E-79DA6E48EDA1" sourceRef="sid-6E0604FC-8D7B-4F99-BE32-76EE9DE7905A" targetRef="sid-9F7131AC-5F2D-4B9B-ABE5-C1ED4DFCFA00"></sequenceFlow>
      <sequenceFlow id="sid-B7138C91-6ED6-4A79-9F16-0C3EA187AD3F" sourceRef="sid-9F7131AC-5F2D-4B9B-ABE5-C1ED4DFCFA00" targetRef="sid-B0303DE4-1024-488D-82CD-78A4A1D2D336"></sequenceFlow>
      <sequenceFlow id="sid-B701E023-89C8-481F-96AF-68D62E040712" sourceRef="sid-B0303DE4-1024-488D-82CD-78A4A1D2D336" targetRef="sid-57226B0E-783D-4FD4-87BB-C228F160AEAF">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${pass == 'canel'}]]></conditionExpression>
      </sequenceFlow>
      <sequenceFlow id="sid-4A05D4AC-C06F-4EBD-B371-67034FBC7F55" sourceRef="sid-B0303DE4-1024-488D-82CD-78A4A1D2D336" targetRef="sid-D185BC86-F922-46B4-93C4-2D9C6559A57D">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${pass == 'success'}]]></conditionExpression>
      </sequenceFlow>
      <sequenceFlow id="sid-6525CF68-D40A-461E-80C1-E19DBA86FFB1" sourceRef="sid-B0303DE4-1024-488D-82CD-78A4A1D2D336" targetRef="sid-6F8681D7-0158-460C-8929-DFB96AC5A217">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${pass == 'error'}]]></conditionExpression>
      </sequenceFlow>
    </transaction>
    <sequenceFlow id="sid-CE5E3449-209C-4FA4-B623-84CE1E4D5B58" sourceRef="sid-F0F15ABD-37D9-4EAC-960B-9E099AFEBC6D" targetRef="sid-AC28574D-44AF-4650-BF00-9AB95E26B123"></sequenceFlow>
    <boundaryEvent id="sid-C52F0179-A84E-4328-8996-69C01D3F372F" name="边界取消" attachedToRef="sid-AC28574D-44AF-4650-BF00-9AB95E26B123" cancelActivity="false">
      <cancelEventDefinition></cancelEventDefinition>
    </boundaryEvent>
    <boundaryEvent id="sid-8A361F6E-ECF3-4EDF-AEB3-11E0C4FBC386" name="边界错误" attachedToRef="sid-AC28574D-44AF-4650-BF00-9AB95E26B123">
      <errorEventDefinition></errorEventDefinition>
    </boundaryEvent>
    <association id="sid-7F11E687-8C92-4A8E-8E0A-167DAA300427" sourceRef="sid-5F643838-FA37-41E9-8F1B-1FEDDAA15B45" targetRef="sid-18A68A51-A66F-4760-B610-A126D4EE9636" associationDirection="None"></association>
    <association id="sid-A1FEA0B1-45D5-4FF9-A470-89C75F7C2E51" sourceRef="sid-5520B272-95AD-4301-AF1D-97FAF2A87E6F" targetRef="sid-A66E0771-CB55-4F3F-9D91-7FF0CE66FEE4" associationDirection="None"></association>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_transactionSubProcess">
    <bpmndi:BPMNPlane bpmnElement="transactionSubProcess" id="BPMNPlane_transactionSubProcess">
      <bpmndi:BPMNShape bpmnElement="sid-F0F15ABD-37D9-4EAC-960B-9E099AFEBC6D" id="BPMNShape_sid-F0F15ABD-37D9-4EAC-960B-9E099AFEBC6D">
        <omgdc:Bounds height="30.0" width="30.0" x="120.0" y="185.5"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-84E39C18-F341-4AE7-840E-213581EA1482" id="BPMNShape_sid-84E39C18-F341-4AE7-840E-213581EA1482">
        <omgdc:Bounds height="28.0" width="28.0" x="975.0" y="186.5"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-614C6EFE-4E6D-4CD3-AB4D-E404CB716F13" id="BPMNShape_sid-614C6EFE-4E6D-4CD3-AB4D-E404CB716F13">
        <omgdc:Bounds height="28.0" width="28.0" x="666.0985362601482" y="450.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-DB9AC289-C0A7-4B15-A443-414784487BC6" id="BPMNShape_sid-DB9AC289-C0A7-4B15-A443-414784487BC6">
        <omgdc:Bounds height="28.0" width="28.0" x="295.055896733921" y="450.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-AC28574D-44AF-4650-BF00-9AB95E26B123" id="BPMNShape_sid-AC28574D-44AF-4650-BF00-9AB95E26B123">
        <omgdc:Bounds height="281.0" width="701.9999999999999" x="210.0" y="60.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-46AEBDA5-3FEF-4EF8-8370-922D206FFC3D" id="BPMNShape_sid-46AEBDA5-3FEF-4EF8-8370-922D206FFC3D">
        <omgdc:Bounds height="30.0" width="30.0" x="249.0" y="115.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-66402E45-A7AD-4C27-8CC3-156F0AA4CEE7" id="BPMNShape_sid-66402E45-A7AD-4C27-8CC3-156F0AA4CEE7">
        <omgdc:Bounds height="80.0" width="100.0" x="321.0" y="90.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-6E0604FC-8D7B-4F99-BE32-76EE9DE7905A" id="BPMNShape_sid-6E0604FC-8D7B-4F99-BE32-76EE9DE7905A">
        <omgdc:Bounds height="80.0" width="100.0" x="456.0" y="90.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-B0303DE4-1024-488D-82CD-78A4A1D2D336" id="BPMNShape_sid-B0303DE4-1024-488D-82CD-78A4A1D2D336">
        <omgdc:Bounds height="40.0" width="40.0" x="726.0" y="110.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-6F8681D7-0158-460C-8929-DFB96AC5A217" id="BPMNShape_sid-6F8681D7-0158-460C-8929-DFB96AC5A217">
        <omgdc:Bounds height="28.0" width="28.0" x="732.0" y="257.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-57226B0E-783D-4FD4-87BB-C228F160AEAF" id="BPMNShape_sid-57226B0E-783D-4FD4-87BB-C228F160AEAF">
        <omgdc:Bounds height="28.0" width="28.0" x="861.0" y="116.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-A66E0771-CB55-4F3F-9D91-7FF0CE66FEE4" id="BPMNShape_sid-A66E0771-CB55-4F3F-9D91-7FF0CE66FEE4">
        <omgdc:Bounds height="80.0" width="100.0" x="456.0" y="212.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-18A68A51-A66F-4760-B610-A126D4EE9636" id="BPMNShape_sid-18A68A51-A66F-4760-B610-A126D4EE9636">
        <omgdc:Bounds height="80.0" width="100.0" x="321.0" y="212.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-D185BC86-F922-46B4-93C4-2D9C6559A57D" id="BPMNShape_sid-D185BC86-F922-46B4-93C4-2D9C6559A57D">
        <omgdc:Bounds height="28.0" width="28.0" x="861.0" y="257.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-9F7131AC-5F2D-4B9B-ABE5-C1ED4DFCFA00" id="BPMNShape_sid-9F7131AC-5F2D-4B9B-ABE5-C1ED4DFCFA00">
        <omgdc:Bounds height="80.0" width="100.0" x="591.0" y="90.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-5F643838-FA37-41E9-8F1B-1FEDDAA15B45" id="BPMNShape_sid-5F643838-FA37-41E9-8F1B-1FEDDAA15B45">
        <omgdc:Bounds height="30.0" width="30.0" x="355.86614093745214" y="155.15526788131257"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-5520B272-95AD-4301-AF1D-97FAF2A87E6F" id="BPMNShape_sid-5520B272-95AD-4301-AF1D-97FAF2A87E6F">
        <omgdc:Bounds height="30.0" width="30.0" x="488.25562276244625" y="155.51211766563023"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-7F11E687-8C92-4A8E-8E0A-167DAA300427" id="BPMNShape_sid-7F11E687-8C92-4A8E-8E0A-167DAA300427">
        <omgdc:Bounds height="26.13412934073375" width="0.042743008175307295" x="370.8907834861153" y="185.2223298767445"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-A1FEA0B1-45D5-4FF9-A470-89C75F7C2E51" id="BPMNShape_sid-A1FEA0B1-45D5-4FF9-A470-89C75F7C2E51">
        <omgdc:Bounds height="26.01259281959679" width="0.8760611465018542" x="503.7609718646044" y="185.51728080827144"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-C52F0179-A84E-4328-8996-69C01D3F372F" id="BPMNShape_sid-C52F0179-A84E-4328-8996-69C01D3F372F">
        <omgdc:Bounds height="30.0" width="30.0" x="665.0985362601482" y="326.37475528756636"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-8A361F6E-ECF3-4EDF-AEB3-11E0C4FBC386" id="BPMNShape_sid-8A361F6E-ECF3-4EDF-AEB3-11E0C4FBC386">
        <omgdc:Bounds height="30.0" width="30.0" x="294.055896733921" y="326.09840244348015"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-CE5E3449-209C-4FA4-B623-84CE1E4D5B58" id="BPMNEdge_sid-CE5E3449-209C-4FA4-B623-84CE1E4D5B58">
        <omgdi:waypoint x="150.0" y="200.5"></omgdi:waypoint>
        <omgdi:waypoint x="210.0" y="200.5"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-B7138C91-6ED6-4A79-9F16-0C3EA187AD3F" id="BPMNEdge_sid-B7138C91-6ED6-4A79-9F16-0C3EA187AD3F">
        <omgdi:waypoint x="691.0" y="130.0"></omgdi:waypoint>
        <omgdi:waypoint x="726.0" y="130.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-1190457B-AD79-42A1-AF82-D553997E2552" id="BPMNEdge_sid-1190457B-AD79-42A1-AF82-D553997E2552">
        <omgdi:waypoint x="680.0985362601482" y="356.37475528756636"></omgdi:waypoint>
        <omgdi:waypoint x="680.0985362601482" y="450.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-7F11E687-8C92-4A8E-8E0A-167DAA300427" id="BPMNEdge_sid-7F11E687-8C92-4A8E-8E0A-167DAA300427">
        <omgdi:waypoint x="370.8906737717685" y="185.15524781930048"></omgdi:waypoint>
        <omgdi:waypoint x="370.93457902099124" y="212.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-B701E023-89C8-481F-96AF-68D62E040712" id="BPMNEdge_sid-B701E023-89C8-481F-96AF-68D62E040712">
        <omgdi:waypoint x="765.57421875" y="130.42578125"></omgdi:waypoint>
        <omgdi:waypoint x="861.0001059807191" y="130.05447429579488"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-4A05D4AC-C06F-4EBD-B371-67034FBC7F55" id="BPMNEdge_sid-4A05D4AC-C06F-4EBD-B371-67034FBC7F55">
        <omgdi:waypoint x="755.5762081784386" y="140.42379182156134"></omgdi:waypoint>
        <omgdi:waypoint x="865.5515148956283" y="260.6691660921072"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-6525CF68-D40A-461E-80C1-E19DBA86FFB1" id="BPMNEdge_sid-6525CF68-D40A-461E-80C1-E19DBA86FFB1">
        <omgdi:waypoint x="746.4321428571428" y="149.56785714285715"></omgdi:waypoint>
        <omgdi:waypoint x="746.0498217485747" y="257.0000886505175"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-3ED40954-C3F7-4360-BA68-BF1BD9FCDEB2" id="BPMNEdge_sid-3ED40954-C3F7-4360-BA68-BF1BD9FCDEB2">
        <omgdi:waypoint x="279.0" y="130.0"></omgdi:waypoint>
        <omgdi:waypoint x="321.0" y="130.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-A1FEA0B1-45D5-4FF9-A470-89C75F7C2E51" id="BPMNEdge_sid-A1FEA0B1-45D5-4FF9-A470-89C75F7C2E51">
        <omgdi:waypoint x="503.76051172925065" y="185.50361816195144"></omgdi:waypoint>
        <omgdi:waypoint x="504.6528660905469" y="212.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-551B1C78-B4E1-4D45-9544-8D24117D7B91" id="BPMNEdge_sid-551B1C78-B4E1-4D45-9544-8D24117D7B91">
        <omgdi:waypoint x="309.055896733921" y="356.09840244348015"></omgdi:waypoint>
        <omgdi:waypoint x="309.055896733921" y="450.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-EADDE25C-040A-4639-880E-79DA6E48EDA1" id="BPMNEdge_sid-EADDE25C-040A-4639-880E-79DA6E48EDA1">
        <omgdi:waypoint x="556.0" y="130.0"></omgdi:waypoint>
        <omgdi:waypoint x="591.0" y="130.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-1C0737B3-87B2-4C56-8A1F-D6EC2BB9A6C4" id="BPMNEdge_sid-1C0737B3-87B2-4C56-8A1F-D6EC2BB9A6C4">
        <omgdi:waypoint x="421.0" y="130.0"></omgdi:waypoint>
        <omgdi:waypoint x="456.0" y="130.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-588CFB0B-F353-4AD3-93B7-028E85CF20AF" id="BPMNEdge_sid-588CFB0B-F353-4AD3-93B7-028E85CF20AF">
        <omgdi:waypoint x="911.9999999999999" y="200.5"></omgdi:waypoint>
        <omgdi:waypoint x="975.0" y="200.5"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

### 1.2 手动修改流程定义

![image-20200904172026079](/images/activiti6-18/image-20200904172026079.png)

图中的这2处需要导出bpmn文件后手动添加。然后再重新部署流程。

其中4个Java Service任务的监听类，直接复用介绍**边界补偿事件**时创建的补偿类。

![image-20200904173851024](/images/activiti6-18/image-20200904173851024.png)

### 1.3 结束取消

我们在流程设计时设置当pass设置为canel时，便可以触发结束取消事件。我们先发起一个流程实例

![image-20200904164604644](/images/activiti6-18/image-20200904164604644.png)

此时查看流程图：（此处无法显示事务子流程的边框，先这样，后期再找找合适方法，如果哪位大佬有好的生成实时流程图的方法，请一定要教教我。）

![image-20200904164803388](/images/activiti6-18/image-20200904164803388.png)

使用manager签收任务，并办理：

![image-20200904165602593](/images/activiti6-18/image-20200904165602593.png)

![image-20200904165540164](/images/activiti6-18/image-20200904165540164.png)

控制台打印补偿信息如下

```
B补偿~~~null
A补偿~~~null
```

可以看到流程通过了**结束取消事件**，并且触发了补偿。而且被边界取消事件捕获。符合预期。

### 1.4  错误结束

我们来模拟错误结束的情景，发起一个新的流程，然后manager签收任务（和上面一样，不演示了），并审批（pass设置为error）。

![image-20200904170914005](/images/activiti6-18/image-20200904170914005.png)

![image-20200904171027159](/images/activiti6-18/image-20200904171027159.png)

![image-20200904171058872](/images/activiti6-18/image-20200904171058872.png)

 可以看到流程通过了**错误结束事件**，但是并没有触发补偿。而且被边界错误事件捕获。符合预期。

### 1.5事务成功

我们来模拟错误结束的情景，发起一个新的流程，然后manager签收任务（和上面一样，不演示了），并审批（pass设置为success）。

![image-20200904171609700](/images/activiti6-18/image-20200904171609700.png)

![image-20200904171636389](/images/activiti6-18/image-20200904171636389.png)

![image-20200904171732803](/images/activiti6-18/image-20200904171732803.png)

 可以看到流程正常结束，符合预期。

## 2、与ACID事务的关系

要注意不要将BPMN事务子流程与技术（ACID）事务混淆。BPMN事务子流程不是划分技术事务范围的方法。BPMN事务与技术事务有如下区别：

- ACID事务技术上生存期短暂，而BPMN事务可以持续几小时，几天甚至几个月才完成。（考虑一个场景，事务包括的活动中有一个用户任务。通常人的 响应时间要比程序长。或者，在另一个场景下，BPMN事务可能等待某些业务事件发生，像是特定订单的填写完成。）这些操作通常要比更新数据库字段，或者使 用事务队列存储消息，花长得多的时间完成。
- 因为不可能将业务活动的持续时间限定为技术事务的范围，一个BPMN事务通常会生成多个ACID事务。
- 因为一个BPMN事务可以生成多个ACID事务，就不再使用ACID特性。只能使用补偿的方式来间接实现事务的一致性。
- BPMN业务事务也不使用传统方式回滚。因为它生成多个ACID事务，在BPMN事务取消时，部分ACID事务可能已经提交。这样它们没法回滚。

