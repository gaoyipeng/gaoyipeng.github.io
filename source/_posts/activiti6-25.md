---
title: Activiti（25） 流程的挂起、激活
date: 2020-09-19 13:04:29
tags: Activiti
categories: Activiti
description: Activiti（25） 流程的挂起、激活
typora-root-url: ..
password: kiki
---

本节我们介绍流程的挂起、激活。

使用场景：流程发起后，由于某些情况，需要暂停流程，或者流程暂停后再次激活。

* 当流程实例被挂起时，无法通过下一个节点对应的任务id来继续这个流程实例。

* 通过挂起某一特定的流程实例，可以终止当前的流程实例，而不影响到该流程定义的其他流程实例。

* 激活之后可以继续该流程实例，不会对后续任务造成影响。

* 流程挂起时 act_ru_task 的 SUSPENSION_STATE_ 为 2，默认为1

  

## 1、 代码

```java
@PostMapping( "/suspendProcessInstance")
@ApiOperation(value = "挂起、激活流程实例",notes = "挂起、激活流程实例")
public CommonResponse suspendProcessInstance(@RequestParam(value = "processInstanceId", required = true)@ApiParam(value = "流程实例ID" ,required = true)String processInstanceId,
                                             @RequestParam(value = "suspendState", required = true)@ApiParam(value = "状态： 1、激活 2、挂起" ,required = true)String suspendState) {
    processService.suspendProcessInstance(processInstanceId, suspendState);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("操作成功");
}

```

```java
/**
 * 挂起、激活流程实例
 * @param processInstanceId
 * @param suspendState
 */
@Override
public void suspendProcessInstance(String processInstanceId, String suspendState) {
    if ("2".equals(suspendState)) {
        runtimeService.suspendProcessInstanceById(processInstanceId);
    } else if ("1".equals(suspendState)) {
        runtimeService.activateProcessInstanceById(processInstanceId);
    }
}
```

## 2、流程挂起

我们依然使用动态表单-请假流程来演示。此时的流程状态：

![image-20200919135009573](/images/activiti6-25/image-20200919135009573.png)

当前流程到达了人事审批节点，我们使用hruser查看一下当前的待办。

![image-20200919135243738](/images/activiti6-25/image-20200919135243738.png)

可以看到hruser有一条待办。此时发起人忽然发现，流程需要补充一些线下材料，需要暂停流程，执行流程挂起操作：

![image-20200919135423141](/images/activiti6-25/image-20200919135423141.png)

此时再次查看hruser的待办

![image-20200919135500955](/images/activiti6-25/image-20200919135500955.png)

可以看到，待办已经没有了

## 3、流程激活

流程激活很简单

![image-20200919135736808](/images/activiti6-25/image-20200919135736808.png)

查看hruser待办：

![image-20200919135809793](/images/activiti6-25/image-20200919135809793.png)

发现待办又出现了。



