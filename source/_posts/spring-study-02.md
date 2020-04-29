---
title: Spring源码学习（2）：创建测试模块
date: 2020-04-07 10:51:00
tags: Spring
categories: Spring
description: Spring源码学习（2）：创建测试模块
---



这节我们创建一个Gradle模块，用于以后测试代码。

## 新建Gradle模块

File–>New–>Module 新建一个模块

![artifactid.png](/images/spring/02/artifactid.png)
![module.png](/images/spring/02/module.png)

## 添加模块引用

打开新建module下的build.gradle文件，导入以下依赖

```
plugins {
    id 'java'
}

group 'org.springframework'
version '5.1.0.RC1'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile(project(":spring-beans"))
    compile(project(":spring-core"))
    optional(project(":spring-aop"))
    optional(project(":spring-context"))
    optional(project(":spring-oxm"))
    optional("javax.annotation:javax.annotation-api:1.3.2")
    optional("javax.ejb:javax.ejb-api:3.2")
    optional("javax.enterprise.concurrent:javax.enterprise.concurrent-api:1.0")
    optional("javax.inject:javax.inject:1")
    optional("javax.interceptor:javax.interceptor-api:1.2.2")
    optional("javax.money:money-api:1.0.3")
    optional("javax.validation:validation-api:1.1.0.Final")
    optional("javax.xml.ws:jaxws-api:2.3.0")
    optional("org.aspectj:aspectjweaver:${aspectjVersion}")
    optional("org.codehaus.groovy:groovy:${groovyVersion}")
    optional("org.beanshell:bsh:2.0b5")
    optional("joda-time:joda-time:2.10")
    optional("org.hibernate:hibernate-validator:5.4.2.Final")
    optional("org.jetbrains.kotlin:kotlin-reflect:${kotlinVersion}")
    optional("org.jetbrains.kotlin:kotlin-stdlib:${kotlinVersion}")
    testCompile("org.codehaus.groovy:groovy-xml:${groovyVersion}")
    testCompile("org.codehaus.groovy:groovy-jsr223:${groovyVersion}")
    testCompile("org.codehaus.groovy:groovy-test:${groovyVersion}")
    testCompile("org.apache.commons:commons-pool2:2.6.0")
    testCompile("javax.inject:javax.inject-tck:1")
    testRuntime("javax.xml.bind:jaxb-api:2.3.0")
    testRuntime("org.glassfish:javax.el:3.0.1-b08")
    testRuntime("org.javamoney:moneta:1.3")
    testCompile (group: 'junit', name: 'junit', version: '4.12')
}

```
上面我们引入了项目需要的依赖jar包，基本完工，但是还有个地方需要修改，否则无法进行单元测试。

## IdEA Gradle配置

File–>Settings–>Gradle,修改配置如下：

![setting.png](/images/spring/02/setting.png)

## 测试

在test包下新建 Dog.java 和 DogController：

```
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

	public Dog() {

	}

	public Dog(String name, int age) {
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
```
package com.sxdx.controller;

import com.sxdx.entity.Dog;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.springframework.beans.factory.xml.XmlBeanFactory;
import org.springframework.core.io.ClassPathResource;

/**
 * @program: spring
 * @description: dog
 * @author: garnett
 * @create: 2020-04-07 10:33
 **/

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
		System.out.println("默认构造器");
		Dog dog1 = xmlBeanFactory.getBean("dog1", Dog.class);
		dog1.sayHello();
	}

}

```

在 resources 下新建 dog-bean.xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >

	<!-- ====================实例化bean的方式Begin==================== -->
	<!-- 默认构造实例化 -->
	<bean id="dog1" class="com.sxdx.entity.Dog"/>

	<!-- 指定构造器实例化 -->
	<bean id="dog2" class="com.sxdx.entity.Dog">
		<!-- 指定构造器参数 index对应构造器中参数的位置 -->
		<!-- 也可以通过指定参数类型，指定参数名称来注入属性-->
		<constructor-arg index="0" value="哈士奇"/>
		<constructor-arg index="1" value="3"/>
	</bean>
	<!-- ====================实例化bean的方式End==================== -->
</beans>

```

运行DogController.test1 

![test1.png](/images/spring/02/test1.png)

OK,可以了。