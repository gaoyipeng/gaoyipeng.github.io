---
title: Spring源码学习（10）：Spring AOP
date: 2020-04-24 10:10:09
tags: Spring
categories: Spring
description: Spring源码学习（10）：Spring AOP
---

# Spring AOP 和AspectJ

AOP的常见框架有2种：**Spring AOP 、AspectJ**，他俩经常放在一起比较。

AspectJ 是原始的 AOP 技术, 提供完整的Java AOP 解决方案。它更健壮, 但也比 Spring AOP 复杂得多。

Spring AOP依赖的Spring 框架。它支持在运行期基于动态代理的方式将aspect织入目标代码中来实现AOP。但是Spring AOP的切入点支持有限，而且对于static方法和final方法都无法支持aop（因为此类方法无法生成代理类）

同时Spring借鉴了AspectJ，使得在Spring体系中可以使用类似AspectJ的注解来实现AOP功能。但是这并不意味着使用了AspectJ的核心代码。AOP功能的具体实现，还是Spring AOP自己实现的。

Spring AOP和AscpectJ之间的关系：Spring使用了和AscpectJ一样的注解，并使用Aspectj来做切入点解析和匹配。但是spring AOP运行时仍旧是纯的spring AOP,并不依赖于Aspectj的编译器或者织入器。

网上发现一张图，比较全面：

![aop.png](/images/spring/10/aop.png)

![aop.png](I:\Hexo\source\images\spring\10\aop.png)

一般来说，直接使用Spring AOP就可以实现我们大部分的需求了。

# Spring AOP

Spring AOP有2中配置方法：

- 基于XML Schema的配置文件定义,

- 基于@Aspect系列标注定义。

  

## 基于 XML Schema 的AOP

基于XML Schema也有2中方式配置AOP的实现，我们都来了解一下。

- `<aop:aspect>`允许我们在自定义增强方法。（常用）

- `<aop:advisor>`需要实现Spring提供的Advice接口，使用的较少。反正我记不清那么多接口名称~~

  

### `<aop:aspect>`方式配置

- 目标对象

  ```Java
  package com.sxdx.service;
  
  import com.sxdx.entity.User;
  
  public interface UserService {
  	//添加 user
  	public void addUser(User user);
  	//删除 user
  	public void deleteUser(User user);
  }
  
  ```

  ```Java 
  package com.sxdx.service.impl;
  
  import com.sxdx.entity.User;
  import com.sxdx.service.UserService;
  import org.springframework.stereotype.Service;
  
  @Service
  public class UserServiceImpl implements UserService {
  	@Override
  	public void addUser(User user) {
  		System.out.println("添加 user"+user.getName());
  	}
  
  	@Override
  	public void deleteUser(User user) {
  		System.out.println("删除 user"+user.getName());
  		System.out.println("==抛出异常：" + 1 / 0);
  	}
  }
  
  ```

- 增强代码

  ```java
  package com.sxdx.aspect;
  
  import com.sxdx.entity.User;
  import org.aspectj.lang.ProceedingJoinPoint;
  
  /**
   * @program: spring
   * @description:
   * @author: garnett
   * @create: 2020-04-24 17:11
   **/
  
  public class UserAspect {
  
  	/**
  	 * 前置增强
  	 */
  	public void beforeAdvice(User user) {
  		System.out.println("==前置增强，id：" + user.getId() + "，name：" + user.getName());
  	}
  
  	/**
  	 * 后置异常增强
  	 */
  	public void afterExceptionAdvice(User user) {
  		System.out.println("==后置异常增强，id：" + user.getId() + "，name：" + user.getName());
  	}
  
  	/**
  	 * 后置返回增强
  	 */
  	public void afterReturningAdvice(User user) {
  		System.out.println("==后置返回增强，id：" + user.getId() + "，name：" + user.getName());
  	}
  
  	/**
  	 * 后置最终增强
  	 */
  	public void afterAdvice(User user) {
  		System.out.println("==后置最终增强，id：" + user.getId() + "，name：" + user.getName());
  	}
  
  	/**
  	 * 环绕增强
  	 * 注意: 如果环绕增强里通过try catch对异常做了处理,所以当业务方法抛出异常时,后置异常增强不会被执行.
  	 */
  	public Object roundAdvice(ProceedingJoinPoint p, User user) throws Throwable {
  		System.out.println("==环绕增强开始，id：" + user.getId() + "，name：" + user.getName());
  		Object o = null;
           o = p.proceed();
  		System.out.println("==环绕增强结束，id：" + user.getId() + "，name：" + user.getName());
  		return o;
  	}
  
  }
  
  
  ```

- XML Schema的配置

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
  	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
  	   xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
  	   xmlns:context="http://www.springframework.org/schema/context"
  	   xsi:schemaLocation="http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
  
  	<context:component-scan base-package="com.sxdx" />
  
  	<bean id="userAspect" class="com.sxdx.aspect.UserAspect"/>
  	<!--AOP配置（一）proxy-target-class="true" 表示使用cglib代理，否则默认jdk代理 -->
  	<aop:config proxy-target-class="true">
  		<!-- 切入点-->
  		<aop:pointcut id="pointcut" expression="execution(* com.sxdx.service.impl.UserServiceImpl.*(..)) and args(user)"/>
  		<aop:aspect ref="userAspect" order="0">
  			<!--前置增强，在切入点选择的方法之前执行-->
  			<aop:before method="beforeAdvice" pointcut-ref="pointcut" arg-names="user"/>
  			<!--后置异常增强，在切入点选择的方法抛出异常时执行-->
  			<aop:after-throwing method="afterExceptionAdvice" pointcut-ref="pointcut" arg-names="user"/>
  			<!--后置返回增强，在切入点选择的方法正常返回时执行-->
  			<aop:after-returning method="afterReturningAdvice" pointcut-ref="pointcut" arg-names="user"/>
  			<!--后置最终增强，在切入点选择的方法返回时执行，不管是正常返回还是抛出异常都执行-->
  			<aop:after method="afterAdvice" pointcut-ref="pointcut" arg-names="user"/>
  			<!--
  				环绕增强，环绕着在切入点选择的连接点处的方法所执行的通知，可以决定目标方法是否执行，
  				什么时候执行，执行时是否需要替换方法参数，执行完毕是否需要替换返回值
  			-->
  			<aop:around method="roundAdvice" pointcut-ref="pointcut" arg-names="p,user"/>
  		</aop:aspect>
  	</aop:config>
  </beans>
  ```

- 测试

  ```java
  	@Test
  	public void test2() throws Exception {
  		UserService userService = applicationContext.getBean("userServiceImpl", UserServiceImpl.class);
  		User user = new User(1,"二狗子");
  		userService.addUser(user);
  	}
  
  ```

  ```
  ========测试方法开始=======
  
  ==前置增强，id：1，name：二狗子
  ==环绕增强开始，id：1，name：二狗子
  添加 user二狗子
  ==环绕增强参数值：com.sxdx.entity.User@173b9122
  ==环绕增强结束，id：1，name：二狗子
  ==后置最终增强，id：1，name：二狗子
  ==后置返回增强，id：1，name：二狗子
  
  ========测试方法结束=======
  
  ```

  OK，我们可以从打印内容中看出各种通知的执行顺序：

  前置增强(beforeAdvice) > 环绕增强(roundAdvice) > 后置最终增强(afterAdvice) >后置返回增强(afterReturningAdvice)

- 程序异常情况下的执行顺序

  ```java
  	@Test
  	public void test2() throws Exception {
  		UserService userService = applicationContext.getBean("userServiceImpl", UserServiceImpl.class);
  		User user = new User(1,"二狗子");
  		userService.deleteUser(user);//有java.lang.ArithmeticException异常
  	}
  ```

  ```
  ========测试方法开始=======
  
  ==前置增强，id：1，name：二狗子
  ==环绕增强开始，id：1，name：二狗子
  删除 user二狗子
  ==后置最终增强，id：1，name：二狗子
  ==后置异常增强，id：1，name：二狗子
  
  ========测试方法结束=======
  
  
  Disconnected from the target VM, address: '127.0.0.1:56159', transport: 'socket'
  
  java.lang.ArithmeticException: / by zero
  .....
  ```

  此时的执行情况也是一目了然的，就不做介绍了。

  

### `<aop:advisor>`方式配置

**上面我们使用了`<aop:aspect>`来配置自定义切面，除此之外我们还可以使用`<aop:advisor>`来定义。**

- 实现接口

```java
package com.sxdx.aspect;

import org.springframework.aop.AfterReturningAdvice;
import org.springframework.aop.MethodBeforeAdvice;

import java.lang.reflect.Method;

/**
 * @program: spring
 * @description: advisor
 * @author: garnett
 * @create: 2020-05-10 17:10
 **/

public class UserAspectAdvisor implements MethodBeforeAdvice , AfterReturningAdvice {
	@Override
	public void before(Method method, Object[] args, Object target) throws Throwable {
		System.out.println("==前置增强，方法名为" + method.getName());
	}

	@Override
	public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
		System.out.println("==后置增强，方法名为" + method.getName());
	}
}

```

- 配置xml

```xml
	<bean id="userAspectAdvisor" class="com.sxdx.aspect.UserAspectAdvisor"/>
	<aop:config >
		<!-- 切入点-->
		<aop:pointcut id="pointcut" expression="execution(* com.sxdx.service.impl.UserServiceImpl.*(..)) "/>
		<aop:advisor advice-ref="userAspectAdvisor" pointcut-ref="pointcut"/>
	</aop:config>
```



- 测试

  

```java
	@Test
	public void test3() throws Exception {
		UserService userService = (UserService) applicationContext.getBean("userServiceImpl");
		User user = new User(1,"二狗子");
		userService.addUser(user);
	}
```

- 返回值

  

```
==前置增强，方法名为addUser
添加 user二狗子
==后置增强，方法名为addUser
```



**它们的区别：**

- `<aop:aspect>`允许我们在自定义增强方法。例如

​       <aop:before method="beforeAdvice" pointcut-ref="pointcut" arg-names="user"/>

- `<aop:advisor>`需要实现Spring提供的Advice接口，使用的较少。反正我记不清那么多接口名称~~

  



## 基于@Aspect的AOP

声明式AOP是现在的主流，相对于xml配置文件实现方式，更加的优雅。

- 开启@Aspect注解支持

  ```xml
  <aop:aspectj-autoproxy/>
  ```

- 配置类

  - 使用@Aspect代替xml 中的<aop:config >

  - 使用@Before、@AfterThrowing  注解代替xml中的  `<aop:before/>`  `<aop:after-throwing/>`

    

  ```java
  package com.sxdx.aspect;
  
  import com.sxdx.entity.User;
  import org.aspectj.lang.ProceedingJoinPoint;
  import org.aspectj.lang.annotation.*;
  import org.springframework.stereotype.Component;
  
  @Aspect
  @Component
  public class UserAspectAnno {
  	/**
  	 * 前置增强
  	 */
  	@Before("execution(* com.sxdx.service.impl.UserServiceImpl.*(..)) and args(user)")
  	public void beforeAdvice(User user) {
  		System.out.println("==前置增强，id：" + user.getId() + "，name：" + user.getName());
  	}
  
  	/**
  	 * 后置异常增强
  	 */
  	@AfterThrowing("execution(* com.sxdx.service.impl.UserServiceImpl.*(..)) and args(user)")
  	public void afterExceptionAdvice(User user) {
  		System.out.println("==后置异常增强，id：" + user.getId() + "，name：" + user.getName());
  	}
  
  	/**
  	 * 后置返回增强
  	 */
  	@AfterReturning("execution(* com.sxdx.service.impl.UserServiceImpl.*(..)) and args(user)")
  	public void afterReturningAdvice(User user) {
  		System.out.println("==后置返回增强，id：" + user.getId() + "，name：" + user.getName());
  	}
  
  	/**
  	 * 后置最终增强
  	 */
  	@After("execution(* com.sxdx.service.impl.UserServiceImpl.*(..)) and args(user)")
  	public void afterAdvice(User user) {
  		System.out.println("==后置最终增强，id：" + user.getId() + "，name：" + user.getName());
  	}
  
  	/**
  	 * 环绕增强
  	 * 注意: 如果环绕增强里通过try catch对异常做了处理,所以当业务方法抛出异常时,后置异常增强不会被执行.
  	 */
  	@Around("execution(* com.sxdx.service.impl.UserServiceImpl.*(..)) and args(user)")
  	public Object roundAdvice(ProceedingJoinPoint p, User user) throws Throwable {
  		System.out.println("==环绕增强开始，id：" + user.getId() + "，name：" + user.getName());
  		Object o = null;
           o = p.proceed();
  		System.out.println("==环绕增强结束，id：" + user.getId() + "，name：" + user.getName());
  		return o;
  	}
  
  }
  
  ```

  - 测试类

    

  ```java
  	@Test
  	public void test3() throws Exception {
  		UserService userService = (UserService) applicationContext.getBean("userServiceImpl");
  		User user = new User(1,"二狗子");
  		userService.addUser(user);
  	}
  ```

  - 测试结果

  ```
  ==环绕增强开始，id：1，name：二狗子
  ==前置增强，id：1，name：二狗子
  添加 user二狗子
  ==环绕增强结束，id：1，name：二狗子
  ==后置最终增强，id：1，name：二狗子
  ==后置返回增强，id：1，name：二狗子
  ```



OK，关于AOP就记录到这里。

参考：

[https://blog.csdn.net/lyc_liyanchao/article/details/83751392](https://blog.csdn.net/lyc_liyanchao/article/details/83751392)

[https://blog.csdn.net/a128953ad/article/details/50509437](https://blog.csdn.net/a128953ad/article/details/50509437)