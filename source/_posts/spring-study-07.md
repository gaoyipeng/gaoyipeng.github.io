---
title: Spring源码学习（7）：Bean的生命周期
date: 2020-04-12 16:00:29
tags: Spring
categories: Spring
description: Spring源码学习（7）：Bean的生命周期
typora-root-url: ..
---

# Bean的生命周期

`这节呢，我们来学习下spring bean的生命周期。`



## Bean的生命周期流程图

![index.png](/images/spring/07/index.png)

上面是我自己画的流程图，若容器注册了以上各种接口，程序那么将会按照以上的流程进行。下面将仔细讲解各接口作用。

下载地址:

链接：https://pan.baidu.com/s/1WLX8Q_zp0Y7bWJPzeRZVqw 
提取码：b58o

## 加载配置文件

万里长征第一步。Spring需要加载配置文件，就是我们经常说的applicationContext.xml

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
```

## 解析配置文件

这一步是为了将配置文件中的<bean />等标签信息解析成Spring内部的BeanDefinition对象。

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement{......}
```

BeanDefinition继承了AttributeAccessor和BeanMetadataElement接口。

- AttributeAccessor:这是一个属性访问者，它提供了访问属性的能力。

  ```java
  public interface AttributeAccessor {
  	void setAttribute(String name, @Nullable Object value);
  	@Nullable
  	Object getAttribute(String name);
  	@Nullable
  	Object removeAttribute(String name);
  	boolean hasAttribute(String name);
  	String[] attributeNames();
  }
  ```

- BeanMetadataElement：只有一个方法，用来获取元数据元素的配置源对象：

  ```java
  public interface BeanMetadataElement {
  	@Nullable
  	Object getSource();
  }
  ```



`BeanDefinition`接口是Spring对bean的抽象。我们可以看下它包含的方法。

![BeanDefinition.png](/images/spring/07/BeanDefinition.png)

`BeanDefinition`包含了Bean需要的所有信息，这样Spring就可以利用`BeanDefinition`生成Bean了。

## BeanFactoryPostProcessor和BeanPostProcessor对比

----------------------------------------------------------------------------------

此处我们把这2个接口放在一起介绍。这2个接口很相似，但是有一些区别：

- BeanFactoryPostProcessor：

  可以对bean的定义信息（BeanDefinition）进行处理。也就是说，Spring IoC容器允许BeanFactoryPostProcessor在容器实际实例化bean之前读取 BeanDefinition 数据，并且可以修改它。比如修改Property或者scope。

  我们可以配置多个BeanFactoryPostProcessor。你还能通过设置order属性来控制BeanFactoryPostProcessor的执行次序。

- BeanPostProcessor：后置bean处理器

  后置bean处理器，应用在允许自定义修改新的bean实例。此时，IOC已经帮我们生成了对应的Bean对象。我们可以配置多个BeanPostProcessor。你还能通过设置order属性来控制BeanPostProcessor的执行次序。
  
  

|                   对比                   |   **BeanFactoryPostProcessor**   |        BeanPostProcessor         |
| :--------------------------------------: | :------------------------------: | :------------------------------: |
|                 生命周期                 |        Bean实例化完成之前        | Bean实例化完成之后，初始化方法前 |
| 是否可修改bean定义信息（BeanDefinition） |                是                |                否                |
|          是否可修改bean实例信息          |                否                |                是                |
|             是否支持排序接口             |                是                |                是                |
|                 方法级别                 |      ApplicationContext级别      |      ApplicationContext级别      |
|                 实现方式                 | 实现BeanFactoryPostProcessor接口 |    实现BeanPostProcessor接口     |



### BeanFactoryPostProcessor 接口

**现在比较流行注解的方式来配置spring，所以我们只保留applicationContext.xml，不再用xml 注册Bean。**

- applicationContext.xml

  ```xml
  <!--设置扫描目录-->
  <context:component-scan base-package="com.sxdx" />
  ```

- 实体类

  ```java
  package com.sxdx.entity;
  import org.springframework.stereotype.Component;
  //使用注解注入一个名为processor的Bean
  @Component
  public class Processor {
  	private String name;
  	private int age;
  
  	public Processor() {
  	}
  	public Processor(String name, int age) {
  		this.name = name;
  		this.age = age;
  	}
  //get set...
  	@Override
  	public String toString() {
  		return "Processor{" +
  				"name='" + name + '\'' +
  				", age=" + age +
  				'}';
  	}
  }
  ```

- BeanFactoryPostProcessor实现类

  ```Java
  @Component
  public class BeanFactoryPostProcessorOne implements BeanFactoryPostProcessor ,Ordered{
  	@Override
  	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
  		System.out.println("BeanFactoryPostProcessorOne");
  		BeanDefinition dog = beanFactory.getBeanDefinition("processor");
  		//通过BeanDefinition修改bean定义信息
  		if (dog != null){
  			MutablePropertyValues mv = dog.getPropertyValues();
  			mv.add("name","拉布拉多").add("age",22);
  		}
  	}
  
  	@Override
  	public int getOrder() {
  		return 1;
  	}
  }
  
  ```

  ```java
  @Component
  public class BeanFactoryPostProcessorTwo implements BeanFactoryPostProcessor , Ordered {
  	@Override
  	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
  		System.out.println("BeanFactoryPostProcessorTwo");
  
  	}
  	@Override
  	public int getOrder() {
  		return 2;
  	}
  }
  
  ```

​        BeanFactoryPostProcessorTwo存在的意义主要是验证是否支持排序接口。我们实现了Ordered

- 测试类

  ```java
  public class Test1 {
  	private ApplicationContext applicationContext;
  
  	@Before
  	public void initXmlBeanFactory() {
  		System.out.println("\n========测试方法开始=======\n");
  		applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
  	}
  
  	@After
  	public void after() {
  		System.out.println("\n========测试方法结束=======\n");
  	}
  
  	@Test
  	public void test() throws Exception {
  		Processor processor = applicationContext.getBean("processor",Processor.class);
  		System.out.println(processor.toString());
  	}
  }
  
  ```

  打印结果：

  ```
  ========测试方法开始=======
  
  BeanFactoryPostProcessorOne
  BeanFactoryPostProcessorTwo
  Processor{name='拉布拉多', age=22}
  
  ========测试方法结束=======
  ```

注意：

1. 不要在BeanFactoryPostProcessor实现类中使用beanFactory.getBean()。此时尚未实例化。
2. BeanFactoryPostProcessorOne在BeanFactoryPostProcessorTwo前执行，说明了BeanFactoryPostProcessor支持排序接口，且数字越小越先执行。

### BeanPostProcessor 接口

我们在上面代码的基础上做修改。

- BeanPostProcessor实现类

  ```Java
  @Component
  public class BeanPostProcessorOne implements BeanPostProcessor , Ordered {
  
  	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
  		System.out.println("BeanPostProcessorOne");
  		//获取Bean实例，并修改
  		if (beanName.equals("processor")){
  			((Processor)bean).setAge(100);
  		}
  		return bean;
  	}
  
  
  	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
  		return bean;
  	}
  
  	@Override
  	public int getOrder() {
  		return 2;
  	}
  }
  ```

  ```Java
  @Component
  public class BeanPostProcessorTwo implements BeanPostProcessor , Ordered {
  
  	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
  		System.out.println("BeanPostProcessorTwo");
  		return bean;
  	}
  
  
  	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
  		return bean;
  	}
  
  	@Override
  	public int getOrder() {
  		return 1;
  	}
  }
  ```

  

- 测试类

  ```Java
  	@Test
  	public void test() throws Exception {
  		Processor processor = applicationContext.getBean("processor",Processor.class);
  		System.out.println(processor.toString());
  	}
  ```

  

- 测试结果

  ```
  ========测试方法开始=======
  
  BeanFactoryPostProcessorOne
  BeanFactoryPostProcessorTwo
  BeanPostProcessorTwo
  BeanPostProcessorOne
  BeanPostProcessorTwo
  BeanPostProcessorOne
  BeanPostProcessorTwo
  BeanPostProcessorOne
  Processor{name='拉布拉多', age=100}
  
  ========测试方法结束=======
  ```

  **说明：**

  我们可以看到BeanFactoryPostProcessor先执行，验证了BeanFactoryPostProcessor和BeanPostProcessor的生命周期执行顺序。

  BeanPostProcessorTwo在BeanPostProcessorOne前执行，说明了BeanPostProcessor 支持排序接口，且数字越小越先执行。

## InstantiationAwareBeanPostProcessor 接口

-------------------

**InstantiationAwareBeanPostProcessor** 接口本质是**BeanPostProcessor**的子接口，所以实现InstantiationAwareBeanPostProcessor 除了可以实现InstantiationAwareBeanPostProcessor自身的方法，同时也具有了**BeanPostProcessor**的特性。

可以在一个实现类中同时实现 Bean**实例化**和**初始化**前后的业务代码。

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {

	@Nullable
	default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}
    
	default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		return true;
	}
    
	@Nullable
	default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
			throws BeansException {

		return null;
	}

	@Deprecated
	@Nullable
	default PropertyValues postProcessPropertyValues(
			PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {

		return pvs;
	}

}
```

上面可以看到**InstantiationAwareBeanPostProcessor**共有4个方法，其中postProcessPropertyValues已经过期，使用postProcessProperties替代。

该接口除了具有父接口中的两个方法以外还自己额外定义了三个方法。所以该接口一共定义了6个方法。

Instantiation：表示实例化,对象还未生成

Initialization：表示初始化,对象已经生成

|               方法名                | 描述                                                         |
| :---------------------------------: | :----------------------------------------------------------- |
|   postProcessBeforeInitialization   | BeanPostProcessor接口中的方法,在Bean的自定义初始化方法之前执行 |
|   postProcessAfterInitialization    | BeanPostProcessor接口中的方法 在Bean的自定义初始化方法执行完成之后执行 |
|   postProcessBeforeInstantiation    | 自身方法，是最先执行的方法，它在目标对象实例化之前调用，该方法的返回值类型是Object，我们可以返回任何类型的值。由于这个时候目标对象还未实例化，所以这个返回值可以用来代替原本该生成的目标对象的实例(比如代理对象)。如果该方法的返回值代替原本该生成的目标对象，后续只有postProcessAfterInitialization方法会调用，其它方法不再调用；否则按照正常的流程走 |
|    postProcessAfterInstantiation    | 在目标对象实例化之后调用，这个时候对象已经被实例化，但是该实例的属性还未被设置，都是null。因为它的返回值是决定要不要调用postProcessPropertyValues方法的其中一个因素（因为还有一个因素是mbd.getDependencyCheck()）；如果该方法返回false,并且不需要check，那么postProcessProperties就会被忽略不执行；如果返回true，postProcessProperties 就会被执行 |
| postProcessPropertyValues（已废弃） | postProcessProperties代替它                                  |
|        postProcessProperties        | 对属性值进行修改，如果postProcessAfterInstantiation方法返回false，该方法可能不会被调用。可以在该方法内对属性值进行修改 |

我们来代码验证上面表格中的内容：

- 实体类

  ```java
  public class Processor   {
  	private String name;
  	private int age;
  
  	public Processor() {
  		System.out.println("Processor 实例化");
  	}
  
  	public Processor(String name, int age) {
  		this.name = name;
  		this.age = age;
  	}
  	public void initMethod(){
  		System.out.println("初始化方法");
  	}
     //get set
  	@Override
  	public String toString() {
  		return "Processor{" +
  				"name='" + name + '\'' +
  				", age=" + age +
  				'}';
  	}
  
  
  }
  ```

  我们在原先实体类的基础上，新加了initMethod初始化方法。

- 配置类

  ```java
  @Configuration
  public class LifeCycle {
  	@Bean(initMethod = "initMethod")
  	public Processor processor(){
  		return new Processor();
  	}
  }
  ```

  LifeCycle配置类中定义了bean（beanName = "processor"）

- InstantiationAwareBeanPostProcessor实现类

  ```java
  @Component
  public class MyInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {
  	/**
  	 * BeanPostProcessor接口中的方法
  	 * 在Bean的自定义初始化方法之前执行
  	 * Bean对象已经存在了
  	 */
  	@Override
  	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
  		if (beanName.equals("processor")){
  			((Processor)bean).setAge(100);
  			((Processor)bean).setName("特朗普");
  		}
  		// TODO Auto-generated method stub
  		System.out.println(">>postProcessBeforeInitialization"+"-----"+beanName);
  		return bean;
  	}
  	
  	/**
  	 * InstantiationAwareBeanPostProcessor中自定义的方法
  	 * 在方法实例化之前执行  Bean对象还没有创建
  	 */
  	@Nullable
  	public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
  		System.out.println("--->postProcessBeforeInstantiation"+"-----"+beanName);
  		return null;
  	}
  
  	/**
  	 * BeanPostProcessor接口中的方法
  	 * 在Bean的自定义初始化方法执行完成之后执行
  	 * Bean对象已经存在了
  	 */
  	@Override
  	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
  		System.out.println("<<postProcessAfterInitialization"+"-----"+beanName);
  		return bean;
  	}
  
  	/**
  	 * InstantiationAwareBeanPostProcessor中自定义的方法
  	 * 在方法实例化之后执行  Bean对象已经创建出来了
  	 */
  	public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
  		System.out.println("<---postProcessAfterInstantiation"+"-----"+beanName);
  		return true;
  	}
  
  	/**
  	 * InstantiationAwareBeanPostProcessor中自定义的方法
  	 * 可以用来修改Bean中属性的内容
  	 */
  	@Nullable
  	public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
  			throws BeansException {
  		System.out.println("<---postProcessPropertyValues--->"+"-----"+beanName);
  		return null;
  	}
  }
  
  ```

  

- 测试类

  ```java
  public class Test1 {
  
     private ApplicationContext applicationContext;
  
     @Before
     public void initXmlBeanFactory() {
        System.out.println("\n========测试方法开始=======\n");
        applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
     }
  
     @After
     public void after() {
        System.out.println("\n========测试方法结束=======\n");
     }
  
     @Test
     public void test() throws Exception {
        Processor processor = applicationContext.getBean("processor",Processor.class);
        System.out.println(processor.toString());
     }
  }
  ```

-  测试结果（processor Bean的信息）

  ```
  ========测试方法开始=======
  --->postProcessBeforeInstantiation-----processor
  Processor 实例化
  <---postProcessAfterInstantiation-----processor
  <---postProcessPropertyValues--->-----processor
  >>postProcessBeforeInitialization-----processor
  初始化方法
  <<postProcessAfterInitialization-----processor
  Processor{name='特朗普', age=100}
  ========测试方法结束=======
  ```

  通过输出结果，我们可以看到几个方法的执行顺序。


### postProcessBeforeInstantiation方法

**InstantiationAwareBeanPostProcessor** 接口自身的方法，是最先执行的方法，它在目标对象实例化之前调用，该方法的返回值类型是Object，我们可以返回任何类型的值。由于这个时候目标对象还未实例化，所以这个返回值可以用来代替原本该生成的目标对象的实例(比如代理对象)。如果该方法的返回值代替原本该生成的目标对象，后续只有postProcessAfterInitialization方法会调用，其它方法不再调用；否则按照正常的流程走。

我们修改postProcessBeforeInstantiation实现方法，利用cglib动态代理生成对象替换返回值：

```java
@Nullable
	public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		System.out.println("--->postProcessBeforeInstantiation"+"-----"+beanName);

		// 利用cglib动态代理生成对象返回
		if (beanClass == Processor.class) {
			Enhancer e = new Enhancer();
			e.setSuperclass(beanClass);
			e.setCallback(new MethodInterceptor() {
				@Override
				public Object intercept(Object obj, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {

					System.out.println("目标方法执行前:" + method + "\n");
					Object object = methodProxy.invokeSuper(obj, objects);
					System.out.println("目标方法执行后:" + method + "\n");
					return object;
				}
			});
			Processor processor = (Processor) e.create();
			// 返回代理类
			return processor;
		}
		return null;
	}
```



再次执行测试方法：

```
--->postProcessBeforeInstantiation-----processor
Processor 实例化
<<postProcessAfterInitialization-----processor
目标方法执行前:public java.lang.String com.sxdx.entity.Processor.toString()

目标方法执行后:public java.lang.String com.sxdx.entity.Processor.toString()

Processor{name='null', age=0}

```

可以看到只有 postProcessBeforeInstantiation、postProcessAfterInitialization方法被执行了。具体的源码解释可以参考：

[https://cloud.tencent.com/developer/article/1409273](https://cloud.tencent.com/developer/article/1409273)

### postProcessAfterInstantiation和postProcessProperties方法

这个postProcessAfterInstantiation返回值要注意，因为它的返回值是决定要不要调用postProcessProperty方法的其中一个因素（因为还有一个因素是mbd.getDependencyCheck());如果该方法返回false,并且不需要check，那么postProcessProperty就会被忽略不执行；如果返true，postProcessPropertyValues就会被执行。

我们修改如下方法：

```java
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    //注释掉这个，postProcessBeforeInitialization在postProcessAfterInstantiation后执行，会影响我们的测试
    /*
    if (beanName.equals("processor")){
        ((Processor)bean).setAge(100);
        ((Processor)bean).setName("特朗普");
    }*/
    // TODO Auto-generated method stub
    System.out.println(">>postProcessBeforeInitialization"+"-----"+beanName);
    return bean;
}
//返回值需要为true,才会执行postProcessProperties
public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
    System.out.println("<---postProcessAfterInstantiation"+"-----"+beanName);
    return true;
}

@Nullable
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
    throws BeansException {
    System.out.println("<---postProcessPropertyValues--->"+"-----"+beanName);
    //为bean设置属性值
    if(bean instanceof Processor){
        ((Processor) bean).setName("Garnett");
    }
    return pvs;
}
```

返回测试结果：

```
--->postProcessBeforeInstantiation-----processor
Processor 实例化
<---postProcessAfterInstantiation-----processor
<---postProcessPropertyValues--->-----processor
>>postProcessBeforeInitialization-----processor
初始化方法
<<postProcessAfterInitialization-----processor
Processor{name='Garnett', age=0}
```

可以看到我们成功的为Processor设置了name值。

**该方法执行的条件要注意postProcessAfterInstantiation返回true，且postProcessBeforeInstantiation返回null。**



### 总结

1. InstantiationAwareBeanPostProcessor接口继承BeanPostProcessor接口，它内部提供了3个方法，再加上BeanPostProcessor接口内部的2个方法，所以实现这个接口需要实现5个方法。InstantiationAwareBeanPostProcessor接口的主要作用在于目标对象的实例化过程中需要处理的事情，包括实例化对象的前后过程以及实例的属性设置
2. postProcessBeforeInstantiation方法是最先执行的方法，它在目标对象实例化之前调用，该方法的返回值类型是Object，我们可以返回任何类型的值。由于这个时候目标对象还未实例化，所以这个返回值可以用来代替原本该生成的目标对象的实例(比如代理对象)。如果该方法的返回值代替原本该生成的目标对象，后续只有postProcessAfterInitialization方法会调用，其它方法不再调用；否则按照正常的流程走
3. postProcessAfterInstantiation方法在目标对象实例化之后调用，这个时候对象已经被实例化，但是该实例的属性还未被设置，都是null。因为它的返回值是决定要不要调用postProcessPropertyValues方法的其中一个因素（因为还有一个因素是mbd.getDependencyCheck()）；如果该方法返回false,并且不需要check，那么postProcessPropertyValues就会被忽略不执行；如果返回true, postProcessPropertyValues就会被执行
4. postProcessPropertyValues方法对属性值进行修改(这个时候属性值还未被设置，但是我们可以修改原本该设置进去的属性值)。如果postProcessAfterInstantiation方法返回false，该方法可能不会被调用。可以在该方法内对属性值进行修改
5. 父接口BeanPostProcessor的2个方法postProcessBeforeInitialization和postProcessAfterInitialization都是在目标对象被实例化之后，并且属性也被设置之后调用的

## BeanNameAware接口

如果Bean实现了BeanNameAware接口，则调用setBeanName(String name)返回beanName，**该方法不是设置beanName**，而只是让Bean获取自己在BeanFactory配置中的名字。

```java

@Configuration
public class LifeCycle {
	@Bean(initMethod = "initMethod")
	public Processor processor(){
		return new Processor();
	}
}
```

```Java
public class Processor implements BeanNameAware {
	
	//......其余代码不变,Processor完整的代码，放在文章最后贴出

	@Override
	public void setBeanName(String name) {
		System.out.println("BeanNameAware返回BeanName："+name);
	}
}



```

测试方法：

```
	@Before
	public void initXmlBeanFactory() {
		System.out.println("\n========测试方法开始=======\n");
		applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
	}

	@After
	public void after() {
		((ClassPathXmlApplicationContext) applicationContext).close();
		System.out.println("\n========测试方法结束=======\n");
	}

	@Test
	public void test() throws Exception {
		Processor processor = applicationContext.getBean("processor",Processor.class);
		System.out.println(processor.toString());
	}
```



```

========测试方法开始=======

Processor 实例化
BeanNameAware返回BeanName：processor
初始化方法
Processor{name='null', age=0}

========测试方法结束=======
```

这样我们就可以让Bean获取自己在BeanFactory配置中的名字。

## BeanFactoryAware接口

如果Bean实现BeanFactoryAware接口，会回调该接口的setBeanFactory(BeanFactory beanFactory)方法，传入该Bean的BeanFactory，这样该Bean就获得了自己所在的BeanFactory。这样获得了IOC容器，不用使用@Autowired即可获取容器内的Bean.

```java
public class Processor implements BeanNameAware , BeanFactoryAware {
    //......其余代码不变
     
	@Override
	public void setBeanName(String name) {
		System.out.println("BeanNameAware返回BeanName："+name);
	}

	@Override
	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
		beanFactory.getBean("countProxy",CountProxy.class).queryCount();
	}
}
```

```
Processor 实例化
BeanNameAware返回BeanName：processor
查询账户
初始化方法
Processor{name='null', age=0}
```

可以看到，我们通过beanFactory直接获取到了countProxy对象，并调用了countProxy的queryCount()方法，输出了“查询账户”

## ApplicationContextAware接口

如果Bean实现了ApplicationContextAware接口，需要实现该接口的setApplicationContext(ApplicationContext applicationContext)方法，获取applicationContext

```java
public class Processor implements BeanNameAware , BeanFactoryAware, ApplicationContextAware {

	@Override
	public void setBeanName(String name) {
		System.out.println("BeanNameAware返回BeanName："+name);
	}

	@Override
	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
		beanFactory.getBean("countProxy",CountProxy.class).queryCount();
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		applicationContext.getBean("countProxy",CountProxy.class).queryCount();
	}
}

```

```
Processor 实例化
BeanNameAware返回BeanName：processor
查询账户
查询账户
初始化方法
Processor{name='null', age=0}
```

同样返回了“查询账户”。那BeanFactoryAware、ApplicationContextAware这俩接口有什么不同的地方呢？这就要说到ApplicationContext和BeanFactory的区别了。

### **ApplicationContext**和**BeanFactory**的对比

BeanFactory：是Spring里面最底层的接口，常用的实现类是DefaultListableBeanFactory，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。

ApplicationContext：作为BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能。

- 默认初始化所有的Singleton，也可以通过配置取消预初始化。

- 继承MessageSource，因此支持国际化。

- 资源访问，比如访问URL和文件。

- 事件机制。

- 同时加载多个配置文件。

- 以声明式方式启动并创建Spring容器。

  

我们来看看2个对象提供的可调用方法即可发现：applicationContext提供了更多的功能。

![BeanFactory-method.png](/images/spring/07/BeanFactory-method.png)

![applicationContext-method.png](/images/spring/07/applicationContext-method.png)

## InitializingBean接口

InitializingBean接口是Spring为bean提供了初始化方法。另外一种方法就是配置init-method。

```java
public class Processor implements BeanNameAware , BeanFactoryAware, ApplicationContextAware,InitializingBean {
	//省略其他代码
    
    
    public void initMethod(){
		System.out.println("初始化方法");
	}
	@Override
	public void setBeanName(String name) {
		System.out.println("BeanNameAware返回BeanName："+name);
	}

	@Override
	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
		beanFactory.getBean("countProxy",CountProxy.class).queryCount();
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {

		applicationContext.getBean("countProxy",CountProxy.class).queryCount();
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("afterPropertiesSet初始化");
	}
}

```

```
Processor 实例化
BeanNameAware返回BeanName：processor
查询账户
查询账户
afterPropertiesSet初始化
初始化方法
Processor{name='null', age=0}
```

**总结：**

- Spring为bean提供了两种初始化bean的方式，实现InitializingBean接口，实现afterPropertiesSet方法，或者配置init-method指定，两种方式可以同时使用，不建议使用InitializingBean接口，因为它将你的代码紧耦合到 Spring 代码。 一个更好的做法应该是在bean的配置文件属性指定init-method
- **afterPropertiesSet先于initMethod执行**，实现InitializingBean接口是直接调用afterPropertiesSet方法，比通过反射调用init-method指定的方法效率要高一点，但是init-method方式消除了对spring的依赖。
- **如果调用afterPropertiesSet方法时出错，则不调用init-method指定的方法**。

**到此为止，spring中的bean已经可以使用了，这里又涉及到了bean的作用域问题，对于singleton类型的bean，Spring会将其缓存;对于prototype类型的bean，不缓存，每次都创建新的bean的实例**

## DisposableBean接口

DisposableBean，是容器关闭时销毁bean的一种方式。另外一种方法就是配置destroyMethod。

```java
package com.sxdx.entity;

import com.sxdx.proxy.CountProxy;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * @program: spring
 * @description: cc
 * @author: garnett
 * @create: 2020-04-16 16:59
 **/

public class Processor implements BeanNameAware , BeanFactoryAware, ApplicationContextAware,InitializingBean, DisposableBean {

	public void initMethod(){
		System.out.println("初始化方法");
	}

	public void destroyMethod(){
		System.out.println("destroy-method方法被调动了");
	}
	@Override
	public void setBeanName(String name) {
		System.out.println("BeanNameAware返回BeanName："+name);
	}

	@Override
	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
		beanFactory.getBean("countProxy",CountProxy.class).queryCount();
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {

		applicationContext.getBean("countProxy",CountProxy.class).queryCount();
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("afterPropertiesSet初始化");
	}

	@Override
	public void destroy() throws Exception {
		System.out.println("destroy 销毁");
	}
}

```

```java
@Configuration
public class LifeCycle {


	@Bean(initMethod = "initMethod",destroyMethod = "destroyMethod")
	public Processor processor(){
		return new Processor();
	}
}

```

测试：

```
========测试方法开始=======

Processor 实例化
BeanNameAware返回BeanName：processor
查询账户
查询账户
afterPropertiesSet初始化
初始化方法
Processor{name='null', age=0}
destroy 销毁
destroy-method方法被调动了

========测试方法结束=======
```

**总结：**

- 在对象销毁的时候，会去调用`DisposableBean`的`destroy`方法。先执行 `DisposableBean`的`destroy`方法，后执行 `destroy-method`声明的方法。
- 如果一个原型（多例）实现了DisposableBean是没有啥意义的，因为相应的方法根本不会被调用，当然在XML配置文件中指定了destroy方法，也是没有意义的。

最后附上Processor完整代码

```java
public class Processor implements BeanNameAware , BeanFactoryAware, ApplicationContextAware,InitializingBean, DisposableBean {
	private String name;
	private int age;

	public Processor() {
		System.out.println("Processor 实例化");
	}

	public Processor(String name, int age) {
		this.name = name;
		this.age = age;
	}
	public void initMethod(){
		System.out.println("初始化方法");
	}

	public void destroyMethod(){
		System.out.println("destroy-method方法被调动了");
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

	@Override
	public String toString() {
		return "Processor{" +
				"name='" + name + '\'' +
				", age=" + age +
				'}';
	}

	public void sayGood(){
		System.out.println("大家好, 我叫" + getName() + ", 我今年" + getAge() + "岁了");
	}

	@Override
	public void setBeanName(String name) {
		System.out.println("BeanNameAware返回BeanName："+name);
	}

	@Override
	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
		beanFactory.getBean("countProxy",CountProxy.class).queryCount();
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {

		applicationContext.getBean("countProxy",CountProxy.class).queryCount();
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("afterPropertiesSet初始化");
	}

	@Override
	public void destroy() throws Exception {
		System.out.println("destroy 销毁");
	}
}

```



OK，Bean的生命周期学习，就到这里了。与君共勉~~



参考：

[https://blog.csdn.net/fuzhongmin05/article/details/73389779](https://blog.csdn.net/fuzhongmin05/article/details/73389779)

[https://www.cnblogs.com/zrtqsk/p/3735273.html](https://www.cnblogs.com/zrtqsk/p/3735273.html)

[https://www.jianshu.com/p/1dec08d290c1](https://www.jianshu.com/p/1dec08d290c1)

[https://www.jianshu.com/p/1be623493294](https://www.jianshu.com/p/1be623493294)

[https://www.sohu.com/a/166804449_714863](https://www.sohu.com/a/166804449_714863)

[https://cloud.tencent.com/developer/article/1409273](https://cloud.tencent.com/developer/article/1409273)