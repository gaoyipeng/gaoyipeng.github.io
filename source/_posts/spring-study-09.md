---
title: Spring源码学习（9）：面向切面编程（AOP）概念引入
date: 2020-04-21 16:53:03
tags: Spring
categories: Spring
description: Spring源码学习（9）：面向切面编程（AOP）概念引入
---

前面我们学习了动态代理。这章我们接着讲 Spring 的核心概念---AOP，动态代理是AOP的基础。这也是 Spring 框架中最为核心的一个概念。

# AOP基本概念

AOP（Aspect Oriented Programming），通常称为面向切面编程。它利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

**这节内容介绍AOP的设计初衷，以及AOP的术语，否则理解这些抽象概念太难了~~**

## 引入需求

现在有一张表 User,然后我们要在程序中实现对 User 表的增加和删除操作。

要求：增加和删除操作都要记录日志（本例中用System.out.println()代替日志记录）。

- User.java

  ```
  public class User {
  	private int id;
  	private String name;
  
  	public User(int id, String name) {
  		this.id = id;
  		this.name = name;
  	}
  
  	public int getId() {
  		return id;
  	}
  	public void setId(int id) {
  		this.id = id;
  	}
  	public String getName() {
  		return name;
  	}
	public void setName(String name) {
  		this.name = name;
  	}
  }
  
  ```
  
- UserService 接口

  ```
  public interface UserService {
  	//添加 user
  	public void addUser(User user);
  	//删除 user
  	public void deleteUser(User user);
  }
  ```

  

- UserService的实现类

  ```
  @Service
  public class UserServiceImpl implements UserService {
  	@Override
  	public void addUser(User user) {
  		System.out.println("添加 user"+user.getName());
  	}
  
  	@Override
  	public void deleteUser(User user) {
  		System.out.println("删除 user"+user.getName());
  	}
  }
  
  ```

- 日志记录类

  ```
  public class MyLog {
  	//日志记录开始
  	public void before(){
  		System.out.println("日志记录开始");
  	}
  	//提交事务
  	public void after(){
  		System.out.println("日志记录结束");
  	}
  }
  ```

- 测试

  ```
  	@Test
  	public void test() throws Exception {
  		User user = new User(1,"二狗子");
  		UserService userService = applicationContext.getBean("userServiceImpl", UserServiceImpl.class);
  		userService.addUser(user);
  	}
  ```

  ```
  ========测试方法开始=======
  
  添加 user二狗子
  
  ========测试方法结束=======
  ```

  

## 传统实现

上面是正常的调用过程，此时并未实现日志记录，如何实现呢？

```
	@Test
	public void test() throws Exception {
		UserService userService = applicationContext.getBean("userServiceImpl", UserServiceImpl.class);
		User user = new User(1,"二狗子");
		MyLog myLog = new MyLog();
		myLog.before();
		userService.addUser(user);
		myLog.after();
	}
```

```
========测试方法开始=======

日志记录开始
添加 user二狗子
日志记录结束

========测试方法结束=======
```



以上的方法可以实现吗？当然可以，但是如果userService定义了100个方法，并且都要求实现日志记录呢？总不能每个都这么实现吧

## 使用CGlib动态代理实现日志记录

- 新建代理类

  ```
  public class UserCglibProxy implements MethodInterceptor {
  
  	private MyLog myLog;
  
  	private Enhancer enhancer = new Enhancer();
  
  	// 这里的目标类型为Object，则可以接受任意一种参数作为被代理类
  	public Object getInstance(Class clazz, MyLog myLog) {
  		this.myLog = myLog;
  		enhancer.setSuperclass(clazz);
  		enhancer.setCallback(this);
  		// 返回代理对象
  		return enhancer.create();
  	}
  
  
  
  	@Override
  	public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
  		myLog.before();
  		Object result = methodProxy.invokeSuper(o, objects);
  		myLog.after();
  		return result;
  	}
  
  	public MyLog getMyLog() {
  		return myLog;
  	}
  
  	public void setMyLog(MyLog myLog) {
  		this.myLog = myLog;
  	}
  }
  ```

- 测试

  ```
  	@Test
  	public void test() throws Exception {
  		User user = new User(1,"二狗子");
  		MyLog myLog = new MyLog();
  		//动态代理了UserServiceImpl, myLog负责日志记录
  		UserService userService = (UserService) new UserCglibProxy().getInstance(UserServiceImpl.class,myLog);
  		userService.addUser(user);
  	}
  ```

  ```
  ========测试方法开始=======
  
  日志记录开始
  添加 user二狗子
  日志记录结束
  
  ========测试方法结束=======
  ```

  这样一来我们就实现了需求，使用代理的好处也是显而易见的，代理对象能够代理多个目标类，多个目标方法。将那些影响了多个类的公共行为封装到一个可重用模块中。这个就是AOP的雏形了。



## AOP 术语

有了上面的例子，我们就比较好理解下面这些专业术语了：

- target (目标类)：需要被代理的类。例如：UserServiceImpl
- Joinpoint (连接点) :所谓连接点是指那些可能被拦截到的方法。例如：代理类所有的方法（addUser、deleteUser）
- PointCut (切入点)：被选择增强的连接点。例如：addUser()
- advice (通知/增强)：增强代码。例如：MyLog类中的 after、before方法
- Weaving (织入)：是指把增强advice应用到目标对象target来创建新的代理对象proxy的过程.
- proxy (代理类)：通知+切入点
- aspect (切面)：是切入点pointcut和通知advice的结合
- Advisor(通知器)：跟切面一样，也包括通知和切点

## **AOP 的通知类型**

Spring按照通知Advice在目标类方法的连接点位置，可以分为5类

- 前置通知：org.springframework.aop.MethodBeforeAdvice

  `在目标方法执行前实施增强，比如上面例子的 before()方法`

- 后置通知： org.springframework.aop.AfterReturningAdvice

  `在目标方法执行后实施增强，比如上面例子的 after()方法`

- 环绕通知 ：org.aopalliance.intercept.MethodInterceptor

  `在目标方法执行前后实施增强`

- 异常抛出通知： org.springframework.aop.ThrowsAdvice

  `在方法抛出异常后实施增强`

- 引介通知 ：org.springframework.aop.IntroductionInterceptor

  引介增强是一个比较特殊的增强，它不是在目标方法周围织入增强，而是为目标类创建新的方法或属性，所以引介增强的连接点是类级别的，而非方法级别的，Spring允许引入新的接口（必须对应一个实现）到所有被代理对象（目标对象）, 在AOP中表示为“干什么（引入什么）”；

