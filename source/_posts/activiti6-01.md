---
title: Activiti入门-01
date: 2020-05-27 14:35:43
tags: Activiti
categories: Activiti
description: Activiti入门-01
password: kiki
---

> 工作流一直是一个OA系统的灵魂。工作以来一直想要学习一套开源的工作流框架，但是一直因为各种原因耽搁了。今天正式开始学习activiti，记录下学习的过程。与君共勉~~~

# Activiti官网

官网地址：[https://www.activiti.org/](https://www.activiti.org/)，截止2020.05.27，Activiti最新版本是7.1.0.m6

Github地址：[https://github.com/Activiti/Activiti](https://github.com/Activiti/Activiti)

![activiti.org.png](/images/activiti/01/activiti.org.png)

# 七大接口

Activiti引擎提供了7大Service接口，均可以通过ProcessEngine(流程引擎获取)

![servicelist.png](/images/activiti/01/servicelist.png)

# 三大流程设计器

- Eclipse Designer：Eclipse插件，用来绘制activiti流程图

- Activiti Modeler：基于Web的流程设计器，可以实现浏览器绘制activiti流程图

- IDEA actiBPM：IDEA插件，兼容性不是很好，不建议使用