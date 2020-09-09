---
title: Activiti（22） Activiti审批意见管理
date: 2020-09-08 14:53:28
tags: Activiti
categories: Activiti
description: Activiti（22） Activiti审批意见管理
typora-root-url: ..
password: kiki
---

这节我们介绍一下流程审批意见的管理，流程审批时添加审批意见是必须的。我们通过最开始的这个请假流程来完善审批意见管理功能。

![image-20200909170942463](/images/activiti6-22/image-20200909170942463.png)

## 1、为任务创建评论

审批意见一般是在完成审批时添加的，所以这里我们改造一下原先的任务审批接口。

```java
 /**
     * 办理任务，提交task，并保存form
     */
@PostMapping(value = "/task/complete")
@ApiOperation(value = "办理任务",notes = "办理任务、保存form、保存审批意见")
public CommonResponse completeTask(@RequestParam(value = "taskId",required = true) @ApiParam(value = "任务ID",required = true)String taskId,
                                   @RequestParam(value = "processInstanceId",required = false) @ApiParam(value = "流程实例ID",required = false)String processInstanceId,
                                   @RequestParam(value = "comment",required = false,defaultValue = "同意") @ApiParam(value = "审批意见",required = false)String comment,
                                   @RequestParam(value = "type",required = false) @ApiParam(value = "类型",required = false)String type,
                                   HttpServletRequest request) {
    processService.completeTask(taskId,processInstanceId,comment,type,request);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("任务办理成功").data(taskId);
}
```

```java
 @Override
public void completeTask(String taskId, String processInstanceId,String comment,String type,HttpServletRequest request) {
    Map<String, String> formProperties = new HashMap<String, String>();
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
        formService.submitTaskFormData(taskId, formProperties);
    } finally {
        identityService.setAuthenticatedUserId(null);
    }
}
```

## 2、获取任务的审批意见

```java
/**
  * 获取任务的审批意见
 */
@PostMapping(value = "/getTaskComments")
@ApiOperation(value = "获取任务的审批意见",notes = "获取任务的审批意见")
public CommonResponse getTaskComments(@RequestParam(value = "taskId",required = true) @ApiParam(value = "任务ID",required = true)String taskId,
                                      @RequestParam(value = "type",required = false) @ApiParam(value = "类型",required = false)String type,
                                      HttpServletRequest request) {
    List<Comment> taskComments = commentService.getTaskComments(taskId, type);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(taskComments);
}
```

```java
 /**
     * 获取任务的所有审批
     * @param taskId
     * @param type
     * @return
     */
    @Override
    public List<Comment> getTaskComments(String taskId, String type) {
        List<Comment> taskComments = new ArrayList<>();
        if (type == null){
            taskComments = taskService.getTaskComments(taskId);
        }else{
            taskComments = taskService.getTaskComments(taskId,type);
        }
        return taskComments;
    }
```



## 3、通过commentId获取审批意见

```java
/**
     * 通过commentId获取审批意见
     */
    @GetMapping(value = "/getComment/{commentId}")
    @ApiOperation(value = "通过commentId获取审批意见",notes = "通过commentId获取审批意见")
    public CommonResponse getComment(@PathVariable(value = "commentId",required = true) @ApiParam(value = "审批意见ID",required = true)String commentId) {
        Comment comment = commentService.getComment(commentId);
        return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(comment);
    }
```

```java
 /**
     * 通过commentId获取审批意见
     * @param commentId
     * @return
     */
@Override
public Comment 	getComment(String commentId) {
    Comment comment = taskService.getComment(commentId);
    return comment;
}
```



## 4、通过type获取审批意见

```java
/**
     * 通过type获取审批意见
     */
@GetMapping(value = "/getCommentsByType/{type}")
@ApiOperation(value = "通过type获取审批意见",notes = "通过type获取审批意见")
public CommonResponse getCommentsByType(@PathVariable(value = "type",required = true) @ApiParam(value = "审批意见type",required = true)String type) {
    List<Comment> taskComments = commentService.getCommentsByType(type);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(taskComments);
}
```

```java
/**
     * 通过type获取审批意见
     * @param type
     * @return
     */
@Override
public List<Comment> getCommentsByType(String type) {
    List<Comment> taskComments = taskService.getCommentsByType(type);
    return taskComments;
}
```



## 5、获取流程实例的全部审批意见

```java
/**
     * 获取流程实例的全部审批意见
     */
    @PostMapping(value = "/getProcessInstanceComments")
    @ApiOperation(value = "获取流程实例的全部审批意见",notes = "获取流程实例的全部审批意见")
    public CommonResponse getProcessInstanceComments(@RequestParam(value = "processInstanceId",required = true) @ApiParam(value = "流程实例ID",required = true)String processInstanceId,
                                                     @RequestParam(value = "type",required = false) @ApiParam(value = "类型",required = false)String type) {
        List<Comment> taskComments = commentService.getProcessInstanceComments(processInstanceId,type);
        return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(taskComments);
    }
```

```java
 /**
     * 通过某个流程实例的全部审批意见
     * @param processInstanceId
     * @param type
     * @return
     */
@Override
public List<Comment> getProcessInstanceComments(String processInstanceId, String type) {
    List<Comment> taskComments = new ArrayList<>();
    if (type == null){
        taskComments = taskService.getProcessInstanceComments(processInstanceId);
    }else{
        taskComments = taskService.getProcessInstanceComments(processInstanceId,type);
    }
    return taskComments;
}
```



## 6、通过commentId删除审批意见

```java
 /**
     * 通过commentId删除审批意见
     */
@DeleteMapping(value = "/deleteComment/{commentId}")
@ApiOperation(value = "通过commentId删除审批意见",notes = "通过commentId删除审批意见")
public CommonResponse deleteComment(@PathVariable(value = "commentId",required = true) @ApiParam(value = "审批意见ID",required = true)String commentId) {
    commentService.deleteComment(commentId);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("删除成功");
}
```

```java
    /**
     * Removes an individual comment with the given id.
     * @param commentId
     */
    @Override
    public void deleteComment(String commentId) {
        taskService.deleteComment(commentId);
    }
```



## 7、通过commentId、processInstanceId删除审批意见

```java
 /**
     * 通过commentId、processInstanceId删除审批意见
     */
@DeleteMapping(value = "/deleteComments")
@ApiOperation(value = "通过commentId、processInstanceId删除审批意见",notes = "通过commentId、processInstanceId删除审批意见")
public CommonResponse deleteComments(@RequestParam(value = "taskId",required = false) @ApiParam(value = "任务ID",required = false)String taskId,
                                     @RequestParam(value = "processInstanceId",required = false) @ApiParam(value = "流程实例ID",required = false)String processInstanceId) {
    commentService.deleteComments(taskId,processInstanceId);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("删除成功");
}
```

```java
 /**
     * Removes all comments from the provided task and/or process instance
     * @param taskId
     * @param processInstanceId
     */
    @Override
    public void deleteComments(String taskId, String processInstanceId) {
        taskService.deleteComments(taskId,processInstanceId);
    }
```



## 8、综合演示

### 8.1 添加审批意见

我们主要演示审批意见相关的操作，流程的流转操作不熟悉的可以参考动态表单章节。我们直接演示添加审批意见：

![image-20200909184227283](/images/activiti6-22/image-20200909184227283.png)

此时的流程图：

![image-20200909184952519](/images/activiti6-22/image-20200909184952519.png)

### 8.2 通过TaskId来获取审批意见

![image-20200909184917054](/images/activiti6-22/image-20200909184917054.png)

可以看到可以获取到审批意见，**注意：此处如果不传type参数，返回数据为空。**

### 8.3  获取流程实例的全部审批意见

![image-20200909184510552](/images/activiti6-22/image-20200909184510552.png)

可以看到，数据返回正确，我们接着审批流程，HR审批（不同意）：

![image-20200909185820794](/images/activiti6-22/image-20200909185820794.png)

![image-20200909185900832](/images/activiti6-22/image-20200909185900832.png)

此时获取全部审批意见如下：

![image-20200909190011222](/images/activiti6-22/image-20200909190011222.png)

### 8.4 通过commentId获取审批意见

前面我们已经可以获取到审批意见，如果想要知道审批意见的详细信息，可以通过以下方式获取：

![image-20200909190455071](/images/activiti6-22/image-20200909190455071.png)

### 8.5 通过type获取审批意见

![image-20200909190552159](/images/activiti6-22/image-20200909190552159.png)

### 8.6 通过commentId删除审批意见

我们此处删除部门经理审批意见：

![image-20200909191126914](/images/activiti6-22/image-20200909191126914.png)

此时再次查看流程的审批意见，可以看到只剩下HR的审批意见了：

![image-20200909191231014](/images/activiti6-22/image-20200909191231014.png)

### 8.6 通过commentId、processInstanceId删除审批意见

此处演示删除部门经理的审批意见：

![image-20200909191403471](/images/activiti6-22/image-20200909191403471.png)

![image-20200909191418908](/images/activiti6-22/image-20200909191418908.png)

可以看到审批意见全部删除了。

注意：此接口如果添加`commentId`，则只是删除对应流程的Id为`commentId`的审批意见。如果只有`processInstanceId`则删除整个流程实例的审批意见。

