---
title: Spring源码学习（8）：Spring中的代理模式
date: 2020-04-18 10:40:31
tags: Spring
categories: Spring
description: Spring中的代理模式
---

代理模式是23种设计模式的一种，他是指一个对象A通过持有另一个对象B，可以具有B同样的行为的模式。

代理模式可分为**静态代理**和**动态代理**两种。而动态代理又有**JDK、CGLIB动态代理**。

静态代理与动态代理的区别主要在：

- 静态代理在编译时就已经实现，编译完成后代理类是一个实际的class文件
- 动态代理是在运行时动态生成的，即编译完成后没有实际的class文件，而是在运行时动态生成类字节码，并加载到JVM中

# 静态代理

这个较为简单，我们写个小例子。

- 被代理接口和实现类

  ```java
  package com.sxdx.proxy;
  
  public interface Count {
  	// 查询账户
  	void queryCount();
  
  	// 修改账户
  	void updateCount();
  }
  
  ```

  ```java
  
  @Service
  public class CountImpl implements Count {
  	@Override
  	public void queryCount() {
  		System.out.println("查询账户");
  	}
  
  	@Override
  	public void updateCount() {
  		System.out.println("修改账户");
  	}
  }
  
  ```

  

- 代理类

  ```java
  @Component
  public class CountProxy implements Count{
  
  	@Autowired
  	@Qualifier("countImpl")
  	private Count countImpl;
  
  	@Override
  	public void queryCount() {
  		countImpl.queryCount();
  	}
  
  	@Override
  	public void updateCount() {
  		countImpl.updateCount();
  	}
  }
  
  ```

  

- 测试及结果

  ```java
  	@Test
  	public void test1() throws Exception {
  		CountProxy countProxy = applicationContext.getBean("countProxy",CountProxy.class);
  		countProxy.queryCount();
  		countProxy.updateCount();
  	}
  ```

  

  ```
  ========测试方法开始=======
  查询账户
  修改账户
  ========测试方法结束=======
  ```

静态代理就是这么简单。静态代理类隐藏了实现类的具体实现，客户端只需知道代理即可。

静态代理类只能为特定的接口(Service)服务。如想要为多个接口服务则需要建立很多个代理类。这样就出现了大量的代码重复。如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。

动态代理才是我们真正需要学习的。

# 动态代理

动态代理分为**JDK动态代理**、**CGLIB动态代理** 2种。

## JDK动态代理

JDK动态代理所用到的代理类在程序调用到代理类对象时才由JVM真正创建，JVM根据传进来的业务实现类对象以及方法名 ，**动态地创建了一个代理类的class文件并被字节码引擎执行，然后通过该代理类对象进行方法调用。**

**JDK动态代理只能代理接口（不支持抽象类），代理类都需要实现InvocationHandler类，实现invoke方法。该invoke方法就是调用被代理接口的方法时需要调用的，该invoke方法返回的值是被代理接口的一个实现类。** 

InvocationHandler是一个接口，只定义了一个方法

```java
public Object invoke(Object proxy, Method method, Object[] args)
```



我们在上面代码的基础上，新建一个JdkProxy代理类

```java
package com.sxdx.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class JdkProxy implements InvocationHandler {

	// 目标对象
	private Object target;

	/**
	 * 构造方法
	 * @param target 目标对象
	 */
	public JdkProxy(Object target) {
		super();
		this.target = target;
	}

	/**
	 * 获取目标对象的代理对象
	 * @return 代理对象
	 */
	public Object getProxy() {
		/**
		 * 参数：
		 * 1、获取当前线程的类加载器
		 * 2、被代理对象的实现接口
		 * 3、调用处理器：这里使用当前对象
		 */
		return Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
				target.getClass().getInterfaces(),
				this);
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("==代理方法开始执行");
		Object invoke = method.invoke(target, args);
		System.out.println("==代理方法结束执行");
		return invoke;
	}
}

```

测试方法及结果：

```java
	@Test
	public void test2() throws Exception {
		JdkProxy handler = new JdkProxy(new CountImpl());
		Count proxy = (Count) handler.getProxy();
		proxy.queryCount();
	}
```

```
==代理方法开始执行
查询账户
==代理方法结束执行
```

打个断点

![jdk-proxy.png](/images/spring/07/jdk-proxy.png)

可以看到 handler.getProxy()返回的是一个CountImpl代理对象。这个代理对象包含写什么内容呢？我们可以使用ProxyGenerator查看

```java
@Test
	public void test2() throws Exception {
		JdkProxy handler = new JdkProxy(new CountImpl());
		Count proxy = (Count) handler.getProxy();
		proxy.queryCount();

		//输出生成的代理类
		String name = "$Proxy";
		byte[] data =
				ProxyGenerator.generateProxyClass(name,new Class[]{Count.class});
		FileOutputStream out =null;
		try {
			out = new FileOutputStream(name+".class");
			System.out.println((new File("hello")).getAbsolutePath());
			out.write(data);
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}finally {
			if(null!=out) try {
				out.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

	}
```

执行代码，项目目录下生成一个ProxyCount.class文件

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

import com.sxdx.proxy.Count;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy extends Proxy implements Count {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m4;
    private static Method m0;

    public $Proxy(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void queryCount() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void updateCount() throws  {
        try {
            super.h.invoke(this, m4, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("com.sxdx.proxy.Count").getMethod("queryCount");
            m4 = Class.forName("com.sxdx.proxy.Count").getMethod("updateCount");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}

```



这就是最终真正的代理类，它继承自Proxy并实现了我们的Count接口。

看到这里 我们就知道了，proxy.queryCount()其实执行的 ProxyCount代理类的queryCount方法

```java
public final void queryCount() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
```



Java动态代理为我们提供了非常灵活的代理机制，但Java动态代理是基于接口的，如果对象没有实现接口我们该如何代理呢？CGLIB登场。

## CGLIB动态代理

CGLIB(Code Generation Library)是一个开源项目！是一个强大的，高性能，高质量的Code生成类库.

**CGLIB特点**

- JDK的动态代理有一个限制，就是使用动态代理的对象必须实现一个或多个接口。
  如果想代理没有实现接口的类，就可以使用CGLIB实现。
- CGLIB是一个强大的高性能的代码生成包，它可以在运行期扩展Java类与实现Java接口。
  它广泛的被许多AOP的框架使用，例如Spring AOP和dynaop，为他们提供方法的interception（拦截）。
- CGLIB包的底层是通过使用一个小而快的字节码处理框架ASM，来转换字节码并生成新的类。
  不鼓励直接使用ASM，因为它需要你对JVM内部结构包括class文件的格式和指令集都很熟悉。

**CGLIB是针对类来实现代理的**，原理是对指定的业务类生成一个子类，并覆盖其中业务方法实现代理。因为采用的是继承，所以不能对final修饰的类进行代理。 在使用的时候需要引入cglib和asm（字节码处理框架）的jar包，**并实现MethodInterceptor接口**。

```
    compile group: 'asm', name: 'asm', version: '3.3.1'
    compile group: 'cglib', name: 'cglib', version: '2.2.2'
```

代理类：

```java
package com.sxdx.proxy;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;


public class CglibProxy implements MethodInterceptor {

	private Enhancer enhancer = new Enhancer();

	// 这里的目标类型为Object，则可以接受任意一种参数作为被代理类
	public Object getInstance(Class clazz) {
		enhancer.setSuperclass(clazz);
		enhancer.setCallback(this);
		// 返回代理对象
		return enhancer.create();
	}


	@Override
	public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
		System.out.println("Cglib代理方法开始执行");
		Object result = methodProxy.invokeSuper(o, objects);
		System.out.println("Cglib代理方法结束执行");
		return result;

	}
}

```

测试类

```java
	@Test
	public void test3() throws Exception {
         //代理了CountImpl
		CountImpl count = (CountImpl) new CglibProxy().getInstance(CountImpl.class);
		count.queryCount();
	}
```

```
Cglib代理方法开始执行
查询账户
Cglib代理方法结束执行
```



OK ，代理模式就先介绍到这里。

参考：

[https://blog.csdn.net/lyc_liyanchao/java/article/details/83655777](https://blog.csdn.net/lyc_liyanchao/java/article/details/83655777)

[https://segmentfault.com/a/1190000011291179?utm_source=tag-newest](https://segmentfault.com/a/1190000011291179?utm_source=tag-newest)



