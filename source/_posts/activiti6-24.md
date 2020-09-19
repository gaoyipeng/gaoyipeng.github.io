---
title: Activiti（24） 任务委托
date: 2020-09-19 12:25:38
tags: Activiti
categories: Activiti
description: Activiti（24） 任务委托
typora-root-url: ..
password: kiki
---

本节我们来介绍一下任务委托。

## 1、任务委托

任务委托，顾名思义，就是把任务委托给别人。在activiti流程中，允许将某个用户的任务委托给另一个用户来处理。

### 1.1 代码

```java
@PostMapping(value = "/delegateTask")
@ApiOperation(value = "委托任务",notes = "委托任务")
public CommonResponse delegateTask(@RequestParam(value = "taskId", required = true)@ApiParam(value = "当前任务ID" ,required = true)String taskId,
                                   @RequestParam(value = "userId", required = true)@ApiParam(value = "委托对象ID" ,required = true)String userId){
    processService.delegateTask(taskId,userId);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("任务委托成功");
}
```

```java
/**
     * 任务委托
     * @param taskId
     * @param userId
     */
@Override
public void delegateTask(String taskId, String userId) {
    taskService.delegateTask(taskId, userId);
    //添加备注
    Task currentTask = taskService.createTaskQuery().taskId(taskId).singleResult();
    taskService.addComment(taskId,currentTask.getProcessInstanceId(),"任务委托给"+userId);
}
```

实现起来非常简单，只需要任务Id和被委托人的userID即可。

### 1.2 演示

我们依然使用动态表单-请假流程来演示。此时的流程状态：

![image-20200919123242678](/images/activiti6-24/image-20200919123242678.png)

#### 1.2.1 获取待办

我们使用hruser获取一下待办：

![image-20200919123359586](/images/activiti6-24/image-20200919123359586.png)

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
        "originalAssignee": null,
        "assignee": null,
        "delegationState": null,
        "parentTaskId": null,
        "name": "人事审批",
        "localizedName": null,
        "description": null,
        "localizedDescription": null,
        "priority": 50,
        "createTime": "2020-09-09 18:32:14",
        "dueDate": null,
        "suspensionState": 1,
        "category": null,
        "executionId": "185014",
        "processInstanceId": "185011",
        "processDefinitionId": "leave-dynamic-from:1:182504",
        "taskDefinitionKey": "hrAudit",
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
        "id": "192506",
        "revision": 1,
        "isInserted": false,
        "isUpdated": false
      }
    ]
  },
  "message": "获取当前人员待办列表成功"
}
```

可以看到当前hruser有一条待办。

#### 1.2.2 委托

我们把任务委托给leaderuser,先来查看下leaderuser的待办：

![image-20200919123631820](/images/activiti6-24/image-20200919123631820.png)

当前leaderuser只有一条待办。接下来我们把任务委托给leaderuser。

![image-20200919124011506](/images/activiti6-24/image-20200919124011506.png)

再次查看leaderuser的待办，我们发现leaderuser多了一条id为192506的待办，委托成功

![image-20200919124146026](/images/activiti6-24/image-20200919124146026.png)

#### 1.2.3 被委托人办理任务

![image-20200919124520375](/images/activiti6-24/image-20200919124520375.png)

我们发现leaderuser办理委托任务时，报错：

```
{
    "message": "Internal server error",
    "exception": "A delegated task cannot be completed, but should be resolved instead."
}
```

原因是被委托的流程需要先resolved这个任务再提交。

我们改造一下任务办理接口:

```java
@Override
public void completeTask(String taskId, String processInstanceId,String comment,String type,HttpServletRequest request) {
    Task task = taskService.createTaskQuery().taskId(taskId).singleResult();

    Map<String, Object> formProperties = new HashMap<String, Object>();
    // 从request中读取参数然后转换
    Map<String, String[]> parameterMap = request.getParameterMap();
    Set<Map.Entry<String, String[]>> entrySet = parameterMap.entrySet();
    for (Map.Entry<String, String[]> entry : entrySet) {
        String key = entry.getKey();
        // fp_的意思是form paremeter
        if (StringUtils.defaultString(key).startsWith("fp_")) {
            formProperties.put(key.split("_")[1], entry.getValue()[0]);
        }
    }
    log.debug("start form parameters: {}", formProperties);
    //TODO 此处应该添加获取当前操作人的代码,先写死用 leaderuser。
    UserQueryImpl user = new UserQueryImpl();
    user = (UserQueryImpl)identityService.createUserQuery().userId(GlobalConfig.getOperator());

    // 用户未登录不能操作，实际应用使用权限框架实现，例如Spring Security、Shiro等
    if (user == null || StringUtils.isBlank(user.getId())) {
        throw new CommonException("用户未登录不能操作");
    }
    try {
        identityService.setAuthenticatedUserId(user.getId());
        //添加评论
        if (!comment.isEmpty()){
            if (type == null){
                taskService.addComment(taskId,processInstanceId,comment);
            }else{
                taskService.addComment(taskId,processInstanceId,type,comment);
            }
        }
        //任务委托处理
        DelegationState delegationState = task.getDelegationState();
        if (delegationState.toString().equals("PENDING")){
            taskService.resolveTask(taskId,formProperties);
        }
        taskService.complete(taskId, formProperties);
    } finally {
        identityService.setAuthenticatedUserId(null);
    }
}
```

内容很多，其实修改比较少，就是以下内容：

我们通过`delegationState`字段来判断该任务是否是委托任务。

```java
Task task = taskService.createTaskQuery().taskId(taskId).singleResult();
Map<String, Object> formProperties = new HashMap<String, Object>();
//任务委托处理
DelegationState delegationState = task.getDelegationState();
if (delegationState.toString().equals("PENDING")){
    taskService.resolveTask(taskId,formProperties);
}
taskService.complete(taskId, formProperties);
```

我们再次执行接口

![image-20200919130302702](/images/activiti6-24/image-20200919130302702.png)

可以看到委托任务办理成功了，此时的流程图如下：

![image-20200919130343179](/images/activiti6-24/image-20200919130343179.png)