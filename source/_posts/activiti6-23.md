---
title: Activiti（23） 流程驳回、流程回退（节点跳转）
date: 2020-09-18 20:09:05
tags: Activiti
categories: Activiti
description: Activiti（23） 流程驳回、流程回退（节点跳转）
typora-root-url: ..
password: kiki
---

由于中国式流程的需要，我们添加一下流程驳回处理，流程驳回分为2种：驳回起点，驳回任意节点。

## 1、Activiti节点跳转的三种方式

### 1.1  流程图配置连线

这个是最简单的实现方式，我们设置好跳转的连线和跳转条件，在驳回时实现跳转是最安全可靠的，不过中国式流程的功能需求(驳回、回退等)，经常是要求在没有连线的情况下完成跳转。如果全部节点都配置驳回连线，那么流程图就会变成一张复杂的蜘蛛网，所以**不采用这种方式**。

### 1.2 动态修改连线

动态修改流程定义环节的连线，然后执行跳转，完成后再恢复流程定义。

这种方法可以实现动态跳转，不需要修改`Activiti`自身执行，但是会动态修改系统中的流程定义缓存对象。理论上这会出现一个多线程下，全局变量不安全的问题。单个`Activiti`流程引擎中，流程定义缓存对象是被所有线程共用的，当一个应用服务器同时收到同个流程定义下的两个不同流程实例、同个环节的任务提交的请求。A任务要求驳回，所以该线程动态修改了流程定义；与此同时，B 任务要求正常流转，但是执行过程中，依据的流程定义已被修改，可能导致B流程也走向了驳回，所以**不采用这种方式**。

### 1.3 修改执行计划

修改当前执行计划，设置从连线流转到目标节点，实现跳转。这种方法即可以实现动态跳转，又没有动态修改流程定义带来的不安全问题。

## 2、流程驳回

流程驳回分为2种：驳回起点，驳回任意节点。

```java
@PostMapping(value = "/rejectAnyNode")
@ApiOperation(value = "流程驳回",notes = "流程驳回")
public CommonResponse rejectAnyNode(
    @RequestParam(value = "taskId", required = true)
    @ApiParam(value = "当前任务ID" ,required = true)String taskId,
    @RequestParam(value = "flowElementId", required = false)
    @ApiParam(value = "驳回指定节点ID(为空则返回流程起始节点)" ,required = false)String flowElementId){
    processService.rejectAnyNode(taskId,flowElementId);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("流程驳回成功");
}
```

```java
/**
 * 驳回指定节点
 * @param taskId
 * @param flowElementId
  */
@Override
@Transactional(noRollbackFor = Exception.class)
public void rejectAnyNode(String taskId, String flowElementId) {

    FlowNode targetNode = null;
    log.info("流程驳回开始>>>>>>>>>>>>>>>>>>>>");
    //获取当前任务
    Task currentTask = taskService.createTaskQuery().taskId(taskId).singleResult();

    //获取当前操作人
    UserQueryImpl user = new UserQueryImpl();
    user = (UserQueryImpl)identityService.createUserQuery().userId(GlobalConfig.getOperator());

 	//判断当前用户是否为该节点处理人
    if(currentTask.getAssignee() == null || !currentTask.getAssignee().equals(user.getId())){
        throw new ActivitiException("当前用户无法驳回,请先签收任务");
    }

    //获取当前节点信息
    String currActivityId = currentTask.getTaskDefinitionKey();
    String processDefinitionId = currentTask.getProcessDefinitionId();
    BpmnModel bpmnModel = repositoryService.getBpmnModel(processDefinitionId);
    FlowNode currFlow = (FlowNode) bpmnModel.getMainProcess().getFlowElement(currActivityId);
    if (null == currFlow) {
        List<SubProcess> subProcessList = bpmnModel.getMainProcess().findFlowElementsOfType(SubProcess.class, true);
        for (SubProcess subProcess : subProcessList) {
            FlowElement flowElement = subProcess.getFlowElement(currActivityId);
            if (flowElement != null) {
                currFlow = (FlowNode) flowElement;
                break;
            }
        }
    }

    //获取流程定义
    Process process = repositoryService.getBpmnModel(currentTask.getProcessDefinitionId()).getMainProcess();
    log.info("流程驳回>>>>>>>,流程名称:{}:{},当前任务:{}:{}",process.getName(),process.getId(),currentTask.getName(),currentTask.getId());
    if (flowElementId == null || flowElementId.isEmpty()){
        //获取流程定义第一个Node节点
        targetNode = findFirstActivityNode(currentTask.getProcessDefinitionId());
    }else{
        //获取目标节点定义
        targetNode = (FlowNode)process.getFlowElement(flowElementId);
    }
    if (targetNode == null){
        throw new ActivitiException("目标节点为空");
    }
    //如果不是同一个流程(子流程)不能驳回
    if (!(currFlow.getParentContainer().equals(targetNode.getParentContainer()))) {
        throw new ActivitiException("不是同一个流程(子流程)不能驳回");
    }
    //删除当前运行任务
    String executionEntityId = managementService.executeCommand(new DeleteTaskCmd(currentTask.getId()));
    //流程执行到来源节点
    managementService.executeCommand(new SetFLowNodeAndGoCmd(targetNode, executionEntityId));
    log.info("流程驳回成功<<<<<<<<<<<<<<<<<<<<<<<");
}


 /**
  * 获得流程定义第一个节点
  */
public FlowNode findFirstActivityNode(String processDefinitionId) {
    BpmnModel model = repositoryService.getBpmnModel(processDefinitionId);
    //Process process = ProcessDefinitionUtil.getProcess(processDefinitionId);
    Process process = model.getMainProcess();
    FlowElement flowElement = process.getInitialFlowElement();
    FlowNode startActivity = (FlowNode) flowElement;

    if (startActivity.getOutgoingFlows().size() != 1) {
        throw new IllegalStateException(
            "start activity outgoing transitions cannot more than 1, now is : "
            + startActivity.getOutgoingFlows().size());
    }

    SequenceFlow sequenceFlow = startActivity.getOutgoingFlows()
        .get(0);
    FlowNode targetActivity = (FlowNode) sequenceFlow.getTargetFlowElement();

    if (!(targetActivity instanceof UserTask)) {
        log.info("first activity is not userTask, just skip");
        return null;
    }

    return targetActivity;
}
```

上面的接口同时实现了驳回起点和驳回任意节点功能，当目标节点**flowElementId**为空时，直接驳回起点，否则驳回**flowElementId**对应的节点，那么**flowElementId**如何获取呢？

我们之前在介绍流程历史管理时，提供了获取流程历史任务、活动的接口，接口返回的**activityId**和**taskDefinitionKey**字段就是所需的**flowElementId**

![image-20200919112133563](/images/activiti6-23/image-20200919112133563.png)

### 2.1 回退任意节点

我们使用请假流程来演示，目前流程处于**销假**节点。我们先演示流程回退到人事审批节点。

![image-20200919113129833](/images/activiti6-23/image-20200919113129833.png)

#### 2.1.1 获取当前操作人任务ID

![image-20200919113559436](/images/activiti6-23/image-20200919113559436.png)

返回：

```json
{
  "code": "0000",
  "data": {
    "pageNum": 1,
    "pageSize": 10,
    "total": 1,
    "pages": 0,
    "firstResult": 0,
    "maxResults": 10,
    "list": [
      {
        "owner": null,
        "originalAssignee": "kafeitu",
        "assignee": "kafeitu",
        "delegationState": null,
        "parentTaskId": null,
        "name": "销假",
        "localizedName": null,
        "description": null,
        "localizedDescription": null,
        "priority": 50,
        "createTime": "2020-09-19 11:30:56",
        "dueDate": null,
        "suspensionState": 1,
        "category": null,
        "executionId": "185014",
        "processInstanceId": "185011",
        "processDefinitionId": "leave-dynamic-from:1:182504",
        "taskDefinitionKey": "reportBack",
        "formKey": null,
        "isDeleted": false,
        "isCanceled": false,
        "eventName": null,
        "currentActivitiListener": null,
        "tenantId": "",
        "queryVariables": null,
        "claimTime": null,
        "usedVariablesCache": {},
        "cachedElContext": null,
        "id": "200007",
        "revision": 1,
        "isInserted": false,
        "isUpdated": false
      }
    ]
  },
  "message": "获取当前人员待办列表成功"
}
```

#### 2.1.2 获取目标节点ID（flowElementId）

flowElementId 为：hrAudit

![image-20200919115037483](/images/activiti6-23/image-20200919115037483.png)

#### 2.1.3 回退

![image-20200919115314199](/images/activiti6-23/image-20200919115314199.png)

此时我们再次查看流程图：

![image-20200919115347424](/images/activiti6-23/image-20200919115347424.png)

可以看到流程已经回退到人事审批节点。



### 2.2 驳回到起点

我们在上图的基础上，继续驳回流程到起点。

说明：此处的所说的流程起点并不是开始节点，而是流程开始后的第一个用户任务，本例中为`部门领导审批节点`。

获取当前操作人任务Id步骤和上面一样，不再演示了。taskId：202502。

注意：驳回是需要先签收任务，否则会报："当前用户无法驳回,请先签收任务"。

#### 2.2.1 回退

![image-20200919120805852](/images/activiti6-23/image-20200919120805852.png)

此时我们再次查看流程图：

![image-20200919120915273](/images/activiti6-23/image-20200919120915273.png)

可以看到流程已经回退到起点了。



