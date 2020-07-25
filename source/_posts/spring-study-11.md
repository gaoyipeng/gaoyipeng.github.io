---
title: Spring源码学习（11）：Spring 事务管理
date: 2020-05-11 16:01:38
tags: Spring
categories: Spring
description: Spring源码学习（11）：Spring 事务管理
typora-root-url: ..
---

# 事务管理

理解事务之前，先讲一个烂大街的例子：取钱。 

​    比如你去ATM机取1000块钱，大体有两个步骤：首先输入密码金额，银行卡扣掉1000元钱；然后ATM出1000元钱。这两个步骤必须是要么都执行要么都不执行。如果银行卡扣除了1000块但是ATM出钱失败的话，你将会损失1000元；如果银行卡扣钱失败但是ATM却出了1000块，那么银行将损失1000元。所以，如果一个步骤成功另一个步骤失败对双方都不是好事，如果不管哪一个步骤失败了以后，整个取钱过程都能回滚，也就是完全取消所有操作的话，这对双方都是极好的。 

   事务就是用来解决类似问题的。事务是一系列的动作，它们综合在一起才是一个完整的工作单元，这些动作必须全部完成，如果有一个失败的话，那么事务就会回滚到最开始的状态，仿佛什么都没发生过一样。 

   在企业级应用程序开发中，事务管理必不可少的技术，用来确保数据的完整性和一致性。

## 事务的四个特性：ACID

- 原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
- 一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
- 隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
- 持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。

## 核心接口

Spring事务管理的实现有许多细节，如果对整个接口框架有个大体了解会非常有利于我们理解事务，下面通过讲解Spring的事务接口来了解Spring实现事务的具体策略。 Spring事务管理涉及的接口的联系如下：

![接口关系](/images/spring/11/interface.png)

这些接口及实现类详情可以参考下图：

![PlatformTransactionManager.png](/images/spring/11/PlatformTransactionManager.png)

### 1、`PlatformTransactionManager`（事务管理器）

Spring并不直接管理事务，而是提供了多种事务管理器，具体的事务管理机制由对应各个平台去实。

Spring事务管理器的接口是org.springframework.transaction.PlatformTransactionManager，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。



**我们以JDBC为例：**

​      如果应用程序中直接使用JDBC来进行持久化，DataSourceTransactionManager会为你处理事务边界。为了可以使用DataSourceTransactionManager，你需要使用如下的XML将其装配到应用程序的上下文定义中：

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
</bean>

```

DataSourceTransactionManager部分代码如下：

```java

@SuppressWarnings("serial")
public class DataSourceTransactionManager extends AbstractPlatformTransactionManager
		implements ResourceTransactionManager, InitializingBean {

	@Nullable
	private DataSource dataSource;

	@Override
	protected void doCommit(DefaultTransactionStatus status) {
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Committing JDBC transaction on Connection [" + con + "]");
		}
		try {
			con.commit();
		}
		catch (SQLException ex) {
			throw new TransactionSystemException("Could not commit JDBC transaction", ex);
		}
	}

	@Override
	protected void doRollback(DefaultTransactionStatus status) {
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Rolling back JDBC transaction on Connection [" + con + "]");
		}
		try {
			con.rollback();
		}
		catch (SQLException ex) {
			throw new TransactionSystemException("Could not roll back JDBC transaction", ex);
		}
	}

}
```

可以看到`DataSourceTransactionManager`实现了`ResourceTransactionManager`接口。提供了事务的具体实现。

### 2、`TransactionDefinition`（事务属性）

![TransactionDefinition.png](/images/spring/11/TransactionDefinition.png)

### 3、`TransactionStatus`（事务状态）

这个接口描述的是一些处理事务提供简单的控制事务执行和查询事务状态的方法，在回滚或提交的时候需要应用对应的事务状态。

![TransactionStatus](/images/spring/11/TransactionStatus.png)



## 事务传播行为

**当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。**

Spring 定义了如下七种传播行为，在上面的`TransactionDefinition`接口中也指明了这七种传播行为，这里以A业务和B业务之间（方法A调用方法B）如何传播事务为例说明：

```java
//伪代码
//A方法（testDo）  B方法（transfer或transferException）
public void testDo(){
    @Transactional(propagation = Propagation.XXX)
    serviceImpl.transfer();
    @Transactional(propagation = Propagation.XXX)
    serviceImpl.transferException();
}
```



　　①、**PROPAGATION_REQUIRED** ：required , 必须。默认值。方法B用 required 修饰。A如果有事务，B将使用该事务；如果A没有事务，B将创建一个新的事务。

- 如果testDo方法添加了事务，则transfer和transferException都加入到 testDo 事务中，属于同一个事务。
- 如果testDo方法**没有添加**事务，则transfer和transferException都创建新的事务，**相互独立，互不干扰。**

------

　　②、**PROPAGATION_REQUIRES_NEW** ：requires_new，必须新的。方法B用 requires_new修饰。如果A有事务，将A的事务挂起，B创建一个新的事务；如果A没有事务，B创建一个新的事务。

- testDo方法**不管有没有添加**事务，则transfer和transferException都创建新的事务，**相互独立，互不干扰**。REQUIRES_NEW是通过开启新的事务实现的，**内部事务和外围事务是两个事务，外围事务回滚不会影响内部事务。**

------

　　③、**PROPAGATION_MANDATORY**：mandatory ，强制。方法B用 mandatory修饰。A如果有事务，B将使用该事务；如果A没有事务，B将抛异常。

------

​		④、**PROPAGATION_SUPPORTS**：supports ，支持。方法B用 supports 修饰。A如果有事务，B将使用该事务；如果A没有事务，B将以非事务执行。

------

　　⑤、**PROPAGATION_NOT_SUPPORTED** ：not_supported ,不支持。如果A有事务，将A的事务挂起，B将以非事务执行；如果A没有事务，B将以非事务执行。

------

　　⑥、**PROPAGATION_NEVER** ：never，从不。如果A有事务，B将抛异常；如果A没有事务，B将以非事务执行。

------

　　⑦、**PROPAGATION_NESTED** ：nested ，嵌套。方法B用 nested  修饰。A和B底层采用保存点机制，形成嵌套事务。

- 如果testDo方法添加了事务，**`Propagation.NESTED`修饰的内部方法属于外部事务的子事务，外围主事务回滚，子事务一定回滚，而内部子事务可以单独回滚而不影响外围主事务和其他子事务。**
- 如果testDo方法**没有添加**事务，**`Propagation.NESTED`和`Propagation.REQUIRED`作用相同**，**内部方法都会新开启自己的事务，且开启的事务相互独立，互不干扰。**

## 事务的隔离级别

> **定义了一个事务可能受其他并发事务影响的程度。**

### **并发事务引起的问题**

　　　　在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务。并发虽然是必须的，但可能会导致以下的问题。

　　　　①、脏读（Dirty reads）——脏读发生在一个事务读取了另一个事务改写但尚未提交的数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。

　　　　②、不可重复读（Nonrepeatable read）——不可重复读发生在一个事务执行相同的查询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。

　　　　③、幻读（Phantom read）——幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录。

**注意：不可重复读重点是修改，而幻读重点是新增或删除。**



### Spring 事务隔离级别

​	   ①、ISOLATION_DEFAULT：（默认）使用数据库默认的隔离级别

　　②、ISOLATION_READ_UNCOMMITTED（读未提交）：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读

　　③、ISOLATION_READ_COMMITTED（读已提交）：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生

　　④、ISOLATION_REPEATABLE_READ（重复读）：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生

　　⑤、ISOLATION_SERIALIZABLE（序列化）：最高的隔离级别，完全服从ACID的隔离级别，确保阻止脏读、不可重复读以及幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的。



### 隔离级别与并发问题对照表

| **隔离级别**               | **脏读** | **不可重复读** | **幻读** |
| -------------------------- | -------- | -------------- | -------- |
| ISOLATION_READ_UNCOMMITTED | 允许     | 允许           | 允许     |
| ISOLATION_READ_COMMITTED   | 不允许   | 允许           | 允许     |
| ISOLATION_REPEATABLE_READ  | 不允许   | 不允许         | 允许     |
| SERIALIZABLE               | 不允许   | 不允许         | 不允许   |



# 代码测试

可以参考，写的很详细，自己懒得写了：[https://segmentfault.com/a/1190000013341344](https://segmentfault.com/a/1190000013341344)

参考：

[https://zhuanlan.zhihu.com/p/88921438](https://zhuanlan.zhihu.com/p/88921438)

[https://www.cnblogs.com/ysocean/p/7617620.html](https://www.cnblogs.com/ysocean/p/7617620.html)

[https://blog.csdn.net/lyc_liyanchao/article/details/85136247](https://blog.csdn.net/lyc_liyanchao/article/details/85136247)

[https://segmentfault.com/a/1190000013341344](https://segmentfault.com/a/1190000013341344)

