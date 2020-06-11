---
title: Activiti（7） 表单分类
date: 2020-06-04 15:54:31
tags: Activiti
categories: Activiti
description: Activiti（7） Activiti表单分类：动态表单、外置表单、普通表单
typora-root-url: ..
password: kiki
---

> 前面几个章节，算是Activiti的入门阶段，让我们知道Activiti的大概工作模式。很多细节并没有介绍，并不足以应对工作中实际的业务。接下来学习表单，从实用的角度学习activiti。

表单：用来显示流程中业务数据的页面和数据。(我是这么理解的~~)

# 1、表单分类

- 动态表单
- 外置表单（FormKey）
- 普通表单

三种表单都有其优缺点，可根据具体场景灵活选用。需要说明的是，三种表单方式只是在任务节点上用户的表单定义方式上面有差别，而流程的运转机制则完全相同。

## 1.1 动态表单

![image-20200608105904854](/images/activiti/activiti6-07/image-20200608105904854.png)

![image-20200608110121083](/images/activiti/activiti6-07/image-20200608110121083.png)

![image-20200608110133758](/images/activiti/activiti6-07/image-20200608110133758.png)

如图所示，区别于普通表单和外置表单，动态表单是直接将工作流节点处的表单嵌入流程定义文件BPMN中，系统利用JS或者模板引擎根据流程定义中表单定义的各个子控件及其属性动态渲染出表单加载出来。

## 1.2 外置表单（FormKey）

这种方式常用于基于工作流平台开发的方式，代码写的很少，开发人员只要把表单内容写好保存到**.form**文件中即可，然后配置每个节点需要的表单名称（form key），实际运行时通过引擎提供的API读取Task对应的form内容输出到页面。

此种方式对于在经常添加新流程的需求比较适用，可以快速发布新流程，把流程设计出来之后再设计表单之后两者关联就可以使用了。

![image-20200608111813077](/images/activiti/activiti6-07/image-20200608111813077.png)

![image-20200608111650312](/images/activiti/activiti6-07/image-20200608111650312.png)

leave-start.form表单内容：

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

**注意：表单的内容都是以key和value的形式数据保存在引擎表中！！！**

## 1.3 普通表单

这个是最灵活的一种方式（但是也最繁琐），常用于业务比较复杂的系统中，或者业务比较固定不变的需求中。

普通表单的特点是把表单的内容存放在一个页面（jsp、jsf、html等）文件中。表单提供流程所需的各种字段，提交到后台后，流程引擎利用这些字段推动流程。

普通表单和和以上两种方式比较有两点区别：

1. **表单**：和第二种外置表单类似，但是表单的显示、表单字段值填充均由开发人员写代码实现。而不是直接读取`.form`的内容。

2. **数据表**：数据表单独设计。而不是和前两种一样把数据以key、value形式保存在引擎表中。（就是说，流程表单的数据不在activit提供的表中存储，我们需要自己设计表来保存）

   

**这节主要就是介绍下表单的分类和基本概念。下一节我们分别介绍这几种表单的具体用法。**