---
title: Spring源码学习（6）：BeanFactory和FactoryBean
date: 2020-04-10 10:28:53
tags: Spring
categories: Spring
description: Spring源码学习（6）：BeanFactory和FactoryBean
typora-root-url: ..
---



## BeanFactory和FactoryBean

BeanFactory是接口，提供了IOC容器最基本的形式，给具体的IOC容器的实现提供了规范。它里面存着很多的bean。

FactoryBean也是接口，为IOC容器中Bean的实现提供了更加灵活的方式，FactoryBean在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式(如果想了解装饰模式参考：[修饰者模式(装饰者模式，Decoration)](https://www.cnblogs.com/aspirant/p/9083082.html) 我们可以在getObject()方法中灵活配置参数的bean。

在Spring源码中有很多FactoryBean的实现类.

**区别**：BeanFactory是个Factory，也就是IOC容器。FactoryBean是个Bean。

在Spring中，**所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的**。但对FactoryBean而言，**这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似**.

### BeanFactory

> 这个其实就是我们前面介绍过的IOC。

BeanFactory接口：IoC容器的顶级接口，是IoC容器的最基础实现，也是访问Spring容器的根接口，负责对bean的创建，访问等工作。

我们前面介绍过的XmlBeanFactory、ApplicationContext都是BeanFactory的子接口，再在其基础之上附加了其他的功能。 

**BeanFacotry是spring中比较原始的Factory。如XmlBeanFactory就是一种典型的BeanFactory。**

原始的BeanFactory无法支持spring的许多插件，**如AOP功能、Web应用等**。ApplicationContext接口,它由BeanFactory接口派生而来，包含BeanFactory的所有功能，所以现在大家都使用ApplicationContext。

BeanFactory的实现体系如下：

![BeanFactory](/images/spring/06/BeanFactory.png)

BeanFactory源码：（很多方法看单词就能猜到它的作用）

```java
public interface BeanFactory {
	String FACTORY_BEAN_PREFIX = "&";
	Object getBean(String name) throws BeansException;
	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
	Object getBean(String name, Object... args) throws BeansException;
	<T> T getBean(Class<T> requiredType) throws BeansException;
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
    //判断工厂中是否包含给定名称的bean定义，若有则返回true
	boolean containsBean(String name);
    //判断给定名称的bean定义是否为单例模式
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
    //判断给定名称的bean定义是否为原型模式
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
    //返回给定bean名称的所有别名
	String[] getAliases(String name);
}
```

### FactoryBean

可以返回bean的实例的工厂bean，通过实现该接口可以对bean进行一些额外的操作，例如根据不同的配置类型返回不同类型的bean，简化xml配置等。

#### FactoryBean 的使用场景

FactoryBean 通常是用来创建比较复杂的bean，一般的bean 直接用xml配置即可，但如果一个bean的创建过程中涉及到很多其他的bean 和复杂的逻辑，用xml配置比较困难，这时可以考虑用FactoryBean。

例如：

FactoryBean的源码：

```java
package org.springframework.beans.factory;
import org.springframework.lang.Nullable;

public interface FactoryBean<T> {
    //返回此工厂管理的对象的实例（可能Singleton和Prototype）如果isSingleton()返回true，则该实例会放到Spring容器中单实例缓存池中
	@Nullable
	T getObject() throws Exception;
    
    
    //返回FactoryBean创建的Bean类型。当配置文件中<bean>的class属性配置的实现类是FactoryBean时，通过getBean()方法返回的不FactoryBean本身
    @Nullable
	Class<?> getObjectType();

    //实例是否单例模式,默认返回true
	default boolean isSingleton() {
		return true;
	}
}

```

**一般情况下，Spring通过反射机制利用的class属性指定实现类实例化Bean，或者使用CGLIB实例化Bean。在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在中提供大量的配置信息。比如需要配置很多的setter。灵活性很差。**

**Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现**。

它们隐藏了实例化一些复杂Bean的细节，给上层应用带来了便利。从Spring3.0开始，FactoryBean开始支持泛型，即接口声明改为FactoryBean<T>的形式。

根据该Bean的beanName从BeanFactory中获取的实际上是FactoryBean的getObject()返回的对象，而不是FactoryBean本身，

如果要获取FactoryBean本身的实例，需要在beanName前面加一个&符号来获取。

我们随便找了一个spring源码中的FactoryBean实现。

```java
public static class BarFactory implements FactoryBean<Bar> {

		@Override
		public Bar getObject() throws Exception {
			return new Bar();
		}

		@Override
		public Class<? extends Bar> getObjectType() {
			return Bar.class;
		}

		@Override
		public boolean isSingleton() {
			return true;
		}

	}
```

Spring已经给我们提供了它的测试类（ Spr6602Tests.java）：

```java

package org.springframework.context.annotation;
import org.junit.Test;
import org.springframework.beans.factory.FactoryBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;

/**
 * Tests to verify that FactoryBean semantics are the same in Configuration
 * classes as in XML.
 *
 * @author Chris Beams
 */
public class Spr6602Tests {

	@Test
	public void testXmlBehavior() throws Exception {
		doAssertions(new ClassPathXmlApplicationContext("Spr6602Tests-context.xml", Spr6602Tests.class));
	}

	@Test
	public void testConfigurationClassBehavior() throws Exception {
		doAssertions(new AnnotationConfigApplicationContext(FooConfig.class));
	}

	private void doAssertions(ApplicationContext ctx) throws Exception {
		Foo foo = ctx.getBean(Foo.class);

		Bar bar1 = ctx.getBean(Bar.class);
		Bar bar2 = ctx.getBean(Bar.class);
		assertThat(bar1, is(bar2));
		assertThat(bar1, is(foo.bar));

		BarFactory barFactory1 = ctx.getBean(BarFactory.class);
		BarFactory barFactory2 = ctx.getBean(BarFactory.class);
		assertThat(barFactory1, is(barFactory2));

		Bar bar3 = barFactory1.getObject();
		Bar bar4 = barFactory1.getObject();
		assertThat(bar3, is(not(bar4)));
		System.out.println(bar3 == bar4);
	}


	@Configuration
	public static class FooConfig {

		@Bean
		public Foo foo() throws Exception {
			return new Foo(barFactory().getObject());
		}

		@Bean
		public BarFactory barFactory() {
			return new BarFactory();
		}
	}


	public static class Foo {

		final Bar bar;

		public Foo(Bar bar) {
			this.bar = bar;
		}
	}


	public static class Bar {
	}


	public static class BarFactory implements FactoryBean<Bar> {

		@Override
		public Bar getObject() throws Exception {
			return new Bar();
		}

		@Override
		public Class<? extends Bar> getObjectType() {
			return Bar.class;
		}

		@Override
		public boolean isSingleton() {
			return true;
		}

	}

}

```

Spr6602Tests-context.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="foo" class="org.springframework.context.annotation.Spr6602Tests$Foo">
		<constructor-arg ref="barFactory" />
	</bean>

	<bean id="barFactory" class="org.springframework.context.annotation.Spr6602Tests$BarFactory" />

</beans>

```

BarFactory利用FactoryBean生成Bar实例。

**这里有个迷惑人的地方：**

BarFactory.isSingleton方法是说明getObject方法返回的是否是同一个对象（单利）。具体的实现却是用户决定的。并不是FactoryBean帮我们实现的。

你可以在isSingleton返回true的同时，getObject每次都返回新的对象，当然这样只会坑到自己。上面的例子就是这样。

虽然配置了isSingleton返回true,但是getObject()每次都返回了一个新的new Bar();

这样导致的结果就是bar3 == bar4 结果为false。

所以如果在isSingleton返回了true,getObject()返回的结果应该实现单利才是合理的。

如果我们想要返回BarFactory的实例，应该在beanName前加&：

```

System.out.println(ctx.getBean("barFactory"));
System.out.println(ctx.getBean("&barFactory"));

```

返回结果分别为：

org.springframework.context.annotation.Spr6602Tests$Bar@1c481ff2
org.springframework.context.annotation.Spr6602Tests$BarFactory@32a68f4f

#### Mybatis对FactoryBean的使用

Mybatis中SqlSessionFactoryBean的部分代码：

```Java


public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {
....
    private SqlSessionFactory sqlSessionFactory;
.....

      public SqlSessionFactory getObject() throws Exception {
        if (this.sqlSessionFactory == null) {
            this.afterPropertiesSet();
        }

        return this.sqlSessionFactory;
    }

    public Class<? extends SqlSessionFactory> getObjectType() {
        return this.sqlSessionFactory == null ? SqlSessionFactory.class : this.sqlSessionFactory.getClass();
    }

    public boolean isSingleton() {
        return true;
    }
  
 
}

```

```xml
<bean id="tradeSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="trade" />
        <property name="mapperLocations" value="classpath*:mapper/trade/*Mapper.xml" />
        <property name="configLocation" value="classpath:mybatis-config.xml" />
        <property name="typeAliasesPackage" value="com.bytebeats.mybatis3.domain.trade" />
    </bean>
```

参考：

[https://www.cnblogs.com/aspirant/p/9082858.html](https://www.cnblogs.com/aspirant/p/9082858.html)

[https://blog.csdn.net/lyc_liyanchao/article/details/82911497](https://blog.csdn.net/lyc_liyanchao/article/details/82911497)

[https://www.jianshu.com/p/b41b0fe2c5ee](https://www.jianshu.com/p/b41b0fe2c5ee)

