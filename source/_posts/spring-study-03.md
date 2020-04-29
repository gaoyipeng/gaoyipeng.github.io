---
title: Spring源码学习（3）：框架介绍
date: 2020-04-07 14:25:06
tags: Spring
categories: Spring
description: Spring 框架介绍
---



## IOC（控制反转）和 DI（依赖注入）

* IOC—Inversion of Control
即“控制反转”，不是什么技术，而是一种设计思想（只可意会不可言传）。在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。
IOC 让程序员不在关注怎么去创建对象，而是关注与对象创建之后的操作，把对象的创建、初始化、销毁等工作交给spring容器来做。

* DI—Dependency Injection
即“依赖注入”：组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。
所有的类都会在spring容器中登记，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。

* IoC和DI的关系
IOC容器帮助我们管理对象，使我们不用频繁的使用new Object()方式来创建对象。降低程序耦合度，既然IOC容器管理了对象的创建，那么创建对象的时候，有可能依赖于其他的对象，即类的属性如何赋值？DI就登场了。
DI通过构造函数注入、Setter方法注入、方法注入（lookup-method、replace-method）将bean注入到需要的组件中。

在Spring中，由Spring IoC容器管理的对象叫做bean。 bean就是由Spring IOC容器实例化、组装和管理的对象。

## 模块

Spring框架的功能被有组织的分散到约20个模块中。这些模块分布在核心容器，数据访问/集成，Web，AOP（面向切面的编程），植入(Instrumentation)，消息传输和测试中，如下面的图所示。

![spring-module.png](/images/spring/03/spring-module.png)

### 核心容器

核心容器由以下模块组成，spring-core， spring-beans，spring-context，springcontext-support，和spring-expression（Spring表达式语言）。

spring-beans 和 spring-context 是Spring框架中IoC容器的基础，模块提供了框架的基础功能，包括IOC和依赖注入功能。

BeanFactory是一个成熟的工厂模式的实现。BeanFactory为Spring的IoC功能提供了底层的基础，但是它仅仅被用于和第三方框架的集成，现在我们更多的使用它的子接口ApplicationContext。

ApplicationContext包括了BeanFactory的所有功能,如果你仅仅只使用简单的BeanFactory，很多的支持功能将不会有效，例如：事务和AOP.

### Web模块

Web层由spring-web，spring-webmvc和spring-websocket 模块组成。