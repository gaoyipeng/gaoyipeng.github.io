---
title: Spring源码学习（4）：Spring IOC（控制反转）
date: 2020-04-08 10:28:22
tags: Spring
categories: Spring
description: Spring IOC（控制反转）
typora-root-url: ..
---

## 基本概念

所谓IoC，对于spring框架来说，就是由spring来负责控制对象的生命周期和对象间的关系。

在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。
IOC 让程序员不在关注怎么去创建对象，而是关注与对象创建之后的操作，把对象的创建、初始化、销毁等工作交给spring容器来做。

## Spring实例化bean的三种方式

- 构造函数实例化
- 静态工厂方法实例化
- 实例工厂方法实例化

在Spring中，由Spring IoC容器管理的对象叫做bean。 bean就是由Spring IOC容器实例化、组装和管理的对象。



### 构造函数实例化

* 创建Dog.java
```java
package com.sxdx.entity;

/**
 * @program: spring
 * @description: dog
 * @author: garnett
 * @create: 2020-04-07 10:29
 **/

public class Dog {
	private String name;

	private int age;

	//默认构造器
	public Dog() {
		System.out.println("默认构造器 init");
	}
	//带参构造器
	public Dog(String name, int age) {
		System.out.println("带参构造器 init");
		this.name = name;
		this.age = age;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public void sayHello() {
		System.out.println("大家好, 我叫" + getName() + ", 我今年" + getAge() + "岁了");
	}
}

```
* 在resources 下创建dog-bean.xml 文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >

	<!-- ====================实例化bean的方式Begin==================== -->
	<!-- 默认构造实例化 -->
	<bean id="dog1" class="com.sxdx.entity.Dog" />

	<!-- 指定构造器实例化 -->
	<bean id="dog2" class="com.sxdx.entity.Dog">
		<!-- 指定ApplicationContext造器参数 name对应构造器中参数名称 -->
		<!-- 也可以通过指定参数类型，指定参数位置来注入属性-->
		<constructor-arg name="name" value="哈士奇"/>
		<constructor-arg name="age" value="3"/>
	</bean>
	<!-- ====================实例化bean的方式End==================== -->
</beans>

```
* 测试类

ApplicationContext测试类

```java
package com.sxdx.controller;

import com.sxdx.entity.Dog;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.springframework.beans.factory.xml.XmlBeanFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.core.io.ClassPathResource;

/**
 * @program: spring
 * @description: dog
 * @author: garnett
 * @create: 2020-04-07 10:33
 **/

public class DogController {

	private ApplicationContext applicationContext;

	@Before
	public void initXmlBeanFactory() {
		System.out.println("\n========测试方法开始=======\n");
		applicationContext = new ClassPathXmlApplicationContext("dog-bean.xml");
	}

	@After
	public void after() {
		System.out.println("\n========测试方法结束=======\n");
	}

	@Test
	public void test1() {
		// 默认构造器
		Dog dog1 = applicationContext.getBean("dog1", Dog.class);
		Dog dog2 = applicationContext.getBean("dog2", Dog.class);
		dog1.sayHello();
		dog2.sayHello();
	}

}

```
XmlBeanFactory测试类
```java
public class DogController {

	private XmlBeanFactory xmlBeanFactory;

    @Before
    public void initXmlBeanFactory() {
        System.out.println("\n========测试方法开始=======\n");
        xmlBeanFactory = new XmlBeanFactory(new ClassPathResource("dog-bean.xml"));
    }
    
    @After
    public void after() {
        System.out.println("\n========测试方法结束=======\n");
    }

	@Test
	public void test1() {
		// 默认构造器
		Dog dog1 = xmlBeanFactory.getBean("dog1", Dog.class);
		Dog dog2 = xmlBeanFactory.getBean("dog2", Dog.class);
		dog1.sayHello();
		dog2.sayHello();
	}

}
```
* 执行测试

XmlBeanFactory与ApplicationContext都是BeanFactory的子接口，实现了IOC功能。

我们执行DogController.test1,控制台打印结果如下：
```
========测试方法开始=======

带参构造器 init
默认构造器 init
大家好, 我叫null, 我今年0岁了
大家好, 我叫哈士奇, 我今年3岁了

========测试方法结束=======
```

区别：ApplicationContext 是在启动spring容器时创建对象，我们在test1()方法开始打一个断点，在执行initXmlBeanFactory()方法后就会打印“带参构造器 init”，证明了我们的这个结论。

但是如果将ApplicationContext换成XmlBeanFactory,依然是在test1()方法中打断点，发现IOC容器在xmlBeanFactory.getBean()后才执行了实例化动作。

如果ApplicationContext想要实现和XmlBeanFactory相同的效果，需要使用 lazy-init="true"

```xml
<bean id="dog1" class="com.sxdx.entity.Dog" lazy-init="true"/>
```

### 静态工厂方法实例化

* 创建静态工厂
```java
package com.sxdx.entity;

/**
 * @program: spring
 * @description: 静态工厂方法
 * @author: garnett
 * @create: 2020-04-08 15:34
 **/

public class DogStaticFactory {

	public DogStaticFactory() {
		System.out.println("静态工厂实例化");
	}

	//静态工厂方法
	public static Dog getInstances(){
		return new Dog();
	}
}


```
* 配置bean

```xml
    <!--
		利用静态工厂方法:
		factory-method：静态工厂类的获取对象的静态方法
		class:静态工厂类的全类名
      -->
	<bean id="dog3" class="com.sxdx.entity.DogStaticFactory" factory-method="getInstances"></bean>
```

* 测试

```
@Test
public void test2() {
    Dog dog3 = applicationContext.getBean("dog3", Dog.class);
    dog3.sayHello();
}


========测试方法开始=======

默认构造器 init
大家好, 我叫null, 我今年0岁了

========测试方法结束=======
```

### 实例工厂方法实例化

* 创建实例工厂
```java 
package com.sxdx.entity;

/**
 * @program: spring
 * @description: 实例工厂方法
 * @author: garnett
 * @create: 2020-04-08 15:46
 **/

public class DogInstanceFactory {
	public DogInstanceFactory() {
		System.out.println("实例工厂方法实例化");
	}

	//静态工厂方法
	public Dog getInstances(){
		return new Dog("德牧",5);
	}
}

```
* 配置bean

```java
	<!--
        利用实例工厂方法:
        factory-bean:指定当前Spring中包含工厂方法的beanId
        factory-method:工厂方法名称
      -->
	<bean id="dogInstanceFactory" class="com.sxdx.entity.DogInstanceFactory"/>
	<bean id="dog4" factory-bean="dogInstanceFactory" factory-method="getInstances"/>
```

* 测试

```java
	@Test
	public void test3() {
		Dog dog4 = applicationContext.getBean("dog4", Dog.class);
		dog4.sayHello();
	}


========测试方法开始=======

实例工厂方法实例化
带参构造器 init
大家好, 我叫德牧, 我今年5岁了

========测试方法结束=======

```
## spring scope说明

spring配置文件中,以下2中写法等效,即默认单利模式
```xml
<bean id="dog1" class="com.sxdx.entity.Dog" />
<bean id="dog1" class="com.sxdx.entity.Dog" scope="singleton"/>
```

scope的常见选项有 singleton、prototype，另外还有request、session、globalSession。

其中singleton，prototype属于Spring bean的基本作用域，request，session，globalSession属于web应用环境的作用域，必须有web应用环境的支持。

另外还有request、session、globalSession

singleton即是我们常说的单利，指在IOC容器中，这个bean只有一个实例。

验证如下：
```
<bean id="dog1" class="com.sxdx.entity.Dog" lazy-init="true" scope="singleton"/>


@Test
public void test1() {
// 默认构造器
Dog dog1 = applicationContext.getBean("dog1", Dog.class);
Dog dog2 = applicationContext.getBean("dog1", Dog.class);

System.out.println(dog1 == dog2);
}
```
返回结果为true。证明dog1和dog2是同一个实例

我们将scope改为prototype（原型模式）：
```
<bean id="dog1" class="com.sxdx.entity.Dog" lazy-init="true" scope="prototype"/>

@Test
public void test1() {
// 默认构造器
Dog dog1 = applicationContext.getBean("dog1", Dog.class);
Dog dog2 = applicationContext.getBean("dog1", Dog.class);

System.out.println(dog1 == dog2);
}
```
返回结果为false。证明dog1和dog2是不是同一个实例。

注意：scope="prototype"时，spring容器启动的时候并不会创建对象，而是在得到 bean 的时候才会创建对象。

