---
title: Activiti（1）-基本概念
date: 2020-05-27 14:35:43
tags: Activiti
categories: Activiti
description: Activiti（1）-基本概念
typora-root-url: ..
---

> 工作流一直是一个OA系统的灵魂。工作以来一直想要学习一套开源的工作流框架，但是一直因为各种原因耽搁了。今天正式开始学习activiti，记录下学习的过程。与君共勉~~~

# 1、Activiti官网

官网地址：[https://www.activiti.org/](https://www.activiti.org/)，截止2020.05.27，Activiti最新版本是7.1.0.m6

Github地址：[https://github.com/Activiti/Activiti](https://github.com/Activiti/Activiti)

![image-20200528094647334](/images/activiti/activiti6-01/image-20200528094647334.png)

# 2、七大接口

Activiti引擎提供了7大Service接口，均可以通过ProcessEngine(流程引擎获取)

![image-20200528094609569](/images/activiti/activiti6-01/image-20200528094609569.png)

# 3、三大流程设计器

- Eclipse Designer(推荐)：Eclipse插件，用来绘制activiti流程图，

  集成教程：https://blog.csdn.net/u012359995/article/details/47753293

- Activiti Modeler(推荐)：基于Web的流程设计器，可以实现浏览器绘制activiti流程图

  ![image-20200528100308803](/images/activiti/activiti6-01/image-20200528100308803.png)

- IDEA actiBPM：IDEA插件，兼容性不是很好，不建议使用

  安装教程：https://www.cnblogs.com/No2-explorer/p/11032469.html
  
  ![image-20200528100400112](/images/activiti/activiti6-01/image-20200528100400112.png)

# 4、28张表

activiti每个版本的表数量不一样，activiti6一共有28张表。

### 表分类

#### 4.1 ACT_RE_XXX

一共 3 张表。

```
ACT_RE_DEPLOYMENT（部署信息表）用来存储部署时需要持久化保存下来的信息。
ACT_RE_MODEL (流程设计模型表) 创建流程的设计模型时，保存在该数据表中。
ACT_RE_PROCDEF（流程定义，解析表）流程解析表，解析成功了，在该表保存一条记录。业务流程定义数据表。
```

#### ACT_RU_XXX

一共 9 张表。

```
ACT_RU_EVENT_SUBSCR （运行时事件）
ACT_RU_EXECUTION（运行时流程执行实例）核心，我的待办任务查询表。
ACT_RU_IDENTITYLINK（身份联系）主要存储当前节点参与者的信息, 任务参与者数据表。
ACT_RU_JOB（运行中的任务）运行时定时任务数据表。
ACT_RU_TASK (运行时任务数据表)（执行中实时任务）待办任务查询表
ACT_RU_VARIABLE (运行时流程变量数据表)
```

![image-20200528113721639](/images/activiti/activiti6-01/image-20200528113721639.png)

**Activiti 6.0.0 新增的 3 张表**

```
ACT_RU_DEADLETTER_JOB
ACT_RU_SUSPENDED_JOB
ACT_RU_TIMER_JOB
```

#### 4.3 ACT_HI_XXX

一共 8 张表。

```
ACT_HI_ACTINST（历史节点表）历史活动信息。这里记录流程流转过的所有节点，与 HI_TASKINST 不同的是，HI_TASKINST 只记录 usertask 内容。
ACT_HI_ATTACHMENT（附件信息）
ACT_HI_COMMENT（历史审批意见表）
ACT_HI_DETAIL（历史详细信息）历史详情表：流程中产生的变量详细，包括控制流程流转的变量，业务表单中填写的流程需要用到的变量等。
ACT_HI_IDENTITYLINK （历史流程人员表）任务参与者数据表。主要存储历史节点参与者的信息。
ACT_HI_PROCINST（历史流程实例信息）核心表
ACT_HI_TASKINST（历史任务流程实例信息）核心表
ACT_HI_VARINST（历史变量信息）
```

#### 4.4 ACT_ID_XXX

一共 4 张表。

```
ACT_ID_INFO（用户扩展信息表）用户扩展信息表。目前该表未用到。
ACT_ID_MEMBERSHIP（用户用户组关联表）用来保存用户的分组信息
ACT_ID_USER（用户信息表）
ACT_ID_GROUP（用户组表）用来存储用户组信息。
```

4.5 其他

一共 4 张表。

```
ACT_GE_BYTEARRAY（通用的流程定义和流程资源）用来保存部署文件的大文本数据。
保存流程定义图片和 xml、Serializable(序列化) 的变量, 即保存所有二进制数据，特别注意类路径部署时候，不要把 svn 等隐藏文件或者其他与流程无关的文件也一起部署到该表中，会造成一些错误（可能导致流程定义无法删除）。

ACT_GE_PROPERTY（系统相关属性）属性数据表。存储这个流程引擎级别的数据。
ACT_EVT_LOG: EVT表示EVENT，目前只有一张表ACT_EVT_LOG，存储事件处理日志，方便管理员跟踪处理。
ACT_PROCDEF_INFO（流程定义扩展表）关联ACTGE_BYTEARRAY与PROC_DEF_ID表。
```

