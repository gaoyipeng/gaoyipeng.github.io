---
title: Spring源码学习（5）：Spring DI（DI依赖注入）
date: 2020-04-09 09:16:59
tags: Spring
categories: Spring
description: Spring DI（DI依赖注入）
typora-root-url: ..
---

上一篇介绍了Spring IOC的基本概念及spring实例化bean的三种方式，这节开始介绍DI（依赖注入）。

IOC容器帮助我们管理对象，使我们不用频繁的使用new Object()方式来创建对象。降低程序耦合度，既然IOC容器管理了对象的创建，那么创建对象的时候，有可能依赖于其他的对象，即类的属性如何赋值？DI就登场了。

DI通过以下几种方法将bean注入到需要的组件(bean)中。

* 构造函数注入
* Setter方法注入
* 方法注入（lookup-method、replace-method）

## 构造函数注入
我们在上节的基础上，新建一个Food.java
```java
package com.sxdx.entity;

/**
 * @program: spring
 * @description: 食物
 * @author: garnett
 * @create: 2020-04-09 09:36
 **/

public class Food {
	private String name ;

	public Food() {

	}

	public Food(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}

```
将他交给spring IOC 管理(此处采用构造器实例化bean)，将会产生一个名为food的bean
```
<bean id="food" class="com.sxdx.entity.Food" >
    <constructor-arg name="name" value="骨头"/>
</bean>
```

想要将food注入到dog中,首先我们为Dog新建一个food属性，并增加一个构造器 Dog(String name, int age, Food food)

```java
public class Dog {
    private String name;

    private int age;

    private Food food;

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

    public Dog(String name, int age, Food food) {
        this.name = name;
        this.age = age;
        this.food = food;
    }
    
   //此处省略get、set 方法

    public void sayHello() {
        System.out.println("大家好, 我叫" + getName() + ", 我今年" + getAge() + "岁了"+"，喜欢吃"+ food.getName());
    }
}
```
通过构造器将food注入
```
<!-- 指定构造器实例化 -->
<bean id="dog2" class="com.sxdx.entity.Dog">
    <constructor-arg name="name" value="哈士奇"/>
    <constructor-arg name="age" value="3"/>
    <!--此处通过构造器，将food注入到dog中-->
    <constructor-arg name="food" ref="food"/>
</bean>
```
测试：
```
@Test
public void test4() {
    Dog dog4 = applicationContext.getBean("dog2", Dog.class);
    dog4.sayHello();
}
```
打印结果：
```
========测试方法开始=======

大家好, 我叫哈士奇, 我今年3岁了，喜欢吃骨头

========测试方法结束=======

```

这样就完成了构造器注入。

## Setter方法注入

除了构造器注入，我们经常使用的还有Setter方法注入。顾名思义，就是需要Java类属性有set方法。

我们在上面代码的基础上，添加setter注入的spring配置信息。

```
	<bean id="dog3" class="com.sxdx.entity.Dog">
		<property name="name" value="德牧"/>
		<property name="age" value="3"/>
		<!--此处通过Setter方法，将food注入到dog中-->
		<property name="food" ref="food"/>
	</bean>
```
通过<property>实现setter方法注入。

测试一下：
```
@Test
public void test4() {
    Dog dog3 = applicationContext.getBean("dog3", Dog.class);
    dog3.sayHello();
}
```
打印输出：
```
========测试方法开始=======

默认构造器 init
大家好, 我叫德牧, 我今年3岁了，喜欢吃骨头

========测试方法结束=======
```
可以看到使用setter方法，使用了默认构造器。以上代码等同于

```
Food food = new Food("骨头");
Dog dog = new Dog();
dog.setName("德牧");
dog.setAge(3);
dog.setFood(food);
dog.sayHello();
```
这样大概就明白了，spring 帮助我们做了什么，IOC 帮助我们new 了food、dog两个bean，DI 则使用setter方法帮我们把food 注入到了dog中。


## 方法注入

以上2种注入方法，已经可以满足大部分的业务场景。
现在有个问题，我们在上一节介绍了bean的scope,有singleton和prototype，其中：

单例模式（singleton）的bean只会被创建一次，IoC容器会缓存该bean实例以供下次使用；

原型模式(prototype) 的bean每次都会创建一个全新的bean，IoC容器不会缓存该bean的实例。

那么如果现在有一个单例模式的bean引用了一个原型模式的bean呢？

若无特殊处理，则被引用的原型模式的bean也会被缓存，这就违背了原型模式的初衷，此时就需要使用lookup-method方法注入来解决该问题。

### lookup-method

 我们对Dog.java 做一些修改

```
package com.sxdx.entity;

/**
 * @program: spring
 * @description: dog
 * @author: garnett
 * @create: 2020-04-07 10:29
 **/

public abstract class Dog {
	private String name;

	private int age;

	private Food food;

    //用于lookup-method注入
	public abstract Food createFood();

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

	public Dog(String name, int age, Food food) {
		this.name = name;
		this.age = age;
		this.food = food;
	}

	//get set...

	public void sayHello() {
		System.out.println("大家好, 我叫" + getName() + ", 我今年" + getAge() + "岁了"+"，喜欢吃"+ food.getName());
	}
}

```

xml 修改：
```
<bean id="dog3" class="com.sxdx.entity.Dog" >
    <property name="name" value="杜牧"/>
    <property name="age" value="3"/>
    <!--此处通过Setter方法，将food注入到dog中-->
    <property name="food" ref="food"/>
    <lookup-method name="createFood" bean="food"/>
</bean>

<bean id="food" class="com.sxdx.entity.Food" scope="prototype">
    <constructor-arg name="name" value="骨头"/>
</bean>
```

测试：
```
@Test
public void test4() {
    Dog dog1 = applicationContext.getBean("dog3", Dog.class);
    Dog dog2 = applicationContext.getBean("dog3", Dog.class);
    //Food food = applicationContext.getBean("food", Food.class);
    System.out.println("Dog : singleton类型，所以 dog1 == dog2 为"+ (dog1 == dog2)); //结果为true,则符合预期


    Food food1 = dog1.getFood();
    Food food2 = dog2.getFood();
    System.out.println("Dog : singleton类型,Food : prototype 类型。未使用lookup-method注入所以 food1 == food2 结果为true,则符合预期:"+ (food1 == food2));


    Food food3 = dog1.createFood();
    Food food4 = dog2.createFood();
    System.out.println("Dog : singleton类型,Food : prototype 类型。使用了lookup-method注入，所以 food3 == food4 结果为false,则符合预期:"+ (food3 == food4));

}
```

打印结果如下：
````
默认构造器 init
Dog : singleton类型，所以 dog1 == dog2 为true
Dog : singleton类型,Food : prototype 类型。未使用lookup-method注入，所以 food1 == food2 结果为true,则符合预期:true
Dog : singleton类型,Food : prototype 类型。使用了lookup-method注入，所以 food3 == food4 结果为false,则符合预期:false

========测试方法结束=======
````

结论：
未使用lookup-method注入时，通过 Dog的实例获取的 Food 实例是被缓存的（虽然配置文件中Food的scope=“prototype”）；
而使用了lookup-method注入时，通过Dog的实例获取的 Food 实例则是每次都是新建的，不是被缓存的，这也就达到了我们的目的。

### replace-method

主要作用就是替换方法体及其返回值，replace-method注入需实现MethodReplacer接口，并重写reimplement方法。


我们在Dog.java中新加如下内容：
```
	public void sayGoodBye() {
		System.out.println("大家好, 我叫" + getName() + ", 我今年" + getAge() + "岁了"+"，GoodBye");
	}
	public void sayGoodBye(String con) {
		System.out.println("大家好, 我叫" + getName() + ", 我今年" + getAge() + "岁了"+"，GoodBye");
	}
```
接下来我们新建一个ReplaceDog.java,负责实现替换的功能。
```java
package com.sxdx.entity;
import org.springframework.beans.factory.support.MethodReplacer;
import java.lang.reflect.Method;
import java.util.Arrays;
/**
 * @program: spring
 * @description: replace-method
 * @author: garnett
 * @create: 2020-04-09 17:27
 **/

public class ReplaceDog implements MethodReplacer {
	@Override
	public Object reimplement(Object obj, Method method, Object[] args) throws Throwable {
		System.out.println("替换内容");
		Arrays.stream(args).forEach(str -> System.out.println("参数:" + str));
		return obj;
	}
}
```

配置replace-method注入：
```
	<bean id="dogReplaceMethod" class="com.sxdx.entity.ReplaceDog" />
	<bean id="dog5" class="com.sxdx.entity.Dog" >
		<replaced-method name="sayGoodBye" replacer="dogReplaceMethod">
			<!-- 区分重载方法, 对应方法名的方法只存在一个时，arg-type将不起作用-->
			<arg-type match="java.lang.String"/>
		</replaced-method>
	</bean>
```
以上配置实现了，使用ReplaceDog.reimplement方法替换Dog.sayGoodBye方法。

测试：
````
	@Test
	public void test4() {
		Dog dog1 = applicationContext.getBean("dog5", Dog.class);
		dog1.sayGoodBye();
		dog1.sayGoodBye("fff");
	}
````

测试结果：
```
========测试方法开始=======

默认构造器 init
大家好, 我叫null, 我今年0岁了，GoodBye
替换内容

========测试方法结束=======
```
说明 dog1.sayGoodBye();没有被替换。dog1.sayGoodBye("fff");被替换了。

这就是replace-method方法注入。

OK，DI介绍完毕。
