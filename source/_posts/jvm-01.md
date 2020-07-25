---
title: JVM初探（1）- 基本概念
date: 2020-06-13 11:36:38
tags: JVM
categories: JVM
description: JVM初探（1）- 基本概念
typora-root-url: ..
---

# 1、JVM介绍

JVM是Java Virtual Machine（Java虚拟机）的缩写，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。

引入Java语言虚拟机后，Java语言在不同平台上运行时不需要重新编译。Java语言使用Java虚拟机屏蔽了与具体平台相关的信息，使得Java语言编译程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。这就是JAVA跨平台的原因。

三种JVM：

- **Sun公司的HotSpot 是目前使用范围最广的Java虚拟机。**

  ![image-20200614095110226](/images/jvm-01/image-20200614095110226.png)

- BEA公司的JRockit（原来的 Bea JRockit）电脑软件，系列产品是一个全面的Java运行时解决方案组合。
- IBM公司的J9 VM 是一个高性能的企业级 Java 虚拟机



# 2、JVM的位置

![image-20200613202133369](/images/jvm-01/image-20200613202133369.png)

# 3、JVM体系结构

我们经常说到的JVM调优，就是发生在堆（Heap）区域。

![image-20200613202700803](/images/jvm-01/image-20200613202700803.png)

# 4、类加载器

## 4.1、类加载器分类

- 虚拟机自带加载器
- 启动类（根）加载器 (BootstrapClassLoader)
- 扩展类加载器(ExtClassLoader)
- 应用程序加载器（AppClassLoader）



![image-20200613204620846](/images/jvm-01/image-20200613204620846.png)

![image-20200613204903455](/images/jvm-01/image-20200613204903455.png)

## 4.2、双亲委派机制

> 当某个类加载器需要加载某个`.class`文件时，它首先把这个任务委托给他的上级类加载器，递归这个操作，如果上级的类加载器没有加载，自己才会去加载这个类。

**双亲委派机制的作用**：

- 1、防止重复加载同一个`.class`。类加载器会一直向上委托，询问上级加载器是否可以加载。如果上级都无法加载才是当前加载器加载。
- 2、保证核心`.class`不能被篡改。通过委托方式，不会去篡改核心`.clas`，即使篡改也不会去加载，即使加载也不会是同一个`.class`对象了。不同的加载器加载同一个`.class`也不是同一个`Class`对象。这样保证了`Class`执行安全。

# 5、Native

Java平台有个用户和本地C代码进行互操作的API，称为Java Native Interface (本地方法接口)。通过Native方法（保存在本地方法栈）即可调用这些JNI接口。

Native常见于两种场景里。一个就是多线程。Java的多线程实际上是调用C语言实现的。再有一个就是硬件开发。涉及到和硬件交互，都会涉及到Native，就是我们说的JNI编程。

# 6、程序计数器

每个线程都有一个程序计数器，属于线程私有。本质是一个指针，指向方法区中的方法字节码（用来存储指向一条指令的地址，也即将要执行的指令代码），了解即可。

# 7、方法区（Method Area）

方法区被所有线程共享，所有类的的字段和方法都保存在这个地方。

**静态变量、常量、类信息（构造方法、接口定义），运行是的常量池也保存在方法区中。但是实例变量存在堆内存中，与方法区无关**

# 8、栈（Stack）

> 栈是一种数据结构。栈的特点：先进后出，后进先出。

举一个好理解的例子，方法栈：

```java
public class Test {
    public static void main(String[] args) {
        new Test().test();
    }
    public void test(){
        System.out.println("");
    }
}
```

Java 会把main方法放在栈底，上面是test方法。main 方法先执行，但是后弹出。

**栈中存放的内容**：8大基本数据类型+对象的引用+实例的方法（方法栈）

# 9、堆 (Heap)

栈和线程是一一对应的，但是一个JVM只有一个堆内存。且内存大小可调节。

堆内存可以细分为3个区域：

- 新生区（伊甸园区 Eden Space）

  ​    伊甸园区 Eden Space：所有对象都是在这里New出来的。

  ​	幸存区form区

  ​    幸存区to区

- 养老区

  当一个对象经历了15次GC（默认）后，就会进入养老区。

- 永久存储区（逻辑上存在，物理上不存在）

  存储java运行时的一些环境或类信息。永久存储区常驻内存中，不会有垃圾回收，只有关闭JVM虚拟机，才会释放这个区域的内存。

  

  **垃圾回收（GC）就发生在新生区、养老区**。

# 10、利用Jprofiler分析内存溢出（OOM）原因

## 1、安装客户端

官网：https://www.ej-technologies.com/download/jprofiler/files，需要破解。

![image-20200614160050397](/images/jvm-01/image-20200614160050397.png)

下面提供了11.1版本的百度云下载地址，包含破解包

链接：https://pan.baidu.com/s/1y0AloyIpWfKdJ6S3ZG_Iyg 
提取码：szvb

我选择了9.2版本，为啥哩，因为9.2版本网上资料多~~，破解包通用。

下载好后安装即可，**保证安装路径无中文、无空格**，我的安装路径是

![image-20200614154911386](/images/jvm-01/image-20200614154911386.png)

生成key:

![image-20200614155255852](/images/jvm-01/image-20200614155255852.png)

具体安装步骤 不记录，百度即可。



## 2、IDEA安装Jprofiler插件

![image-20200614114550691](/images/jvm-01/image-20200614114550691.png)

安装后重启：

![image-20200614121242919](/images/jvm-01/image-20200614121242919.png)

![image-20200614155851483](/images/jvm-01/image-20200614155851483.png)

## 3、测试

![image-20200614155919953](/images/jvm-01/image-20200614155919953.png)

启动项目，就会看到

![image-20200614160006116](/images/jvm-01/image-20200614160006116.png)

## 4、模拟内存溢出

前面3个步骤主要是监控我们的程序，现在我们模拟一个内存溢出的场景；

```java
public class Test {
    public static void main(String[] args) {
        String  str = "aaaaaaaaaaaaaa";
        while(true){
            str+=str;
        }
    }
}
```

配置这个配的JVM信息：-Xms8m -Xmx16m -XX:+HeapDumpOnOutOfMemoryError

初始化jvm内存大小8m,最大16，当发生内存溢出情况，输出一个Dump 文件

![image-20200614161936811](/images/jvm-01/image-20200614161936811.png)

执行程序

![image-20200614162439090](/images/jvm-01/image-20200614162439090.png)

可以看到，发生了内存溢出。并且在根目录下输出了一个`hprof`文件，**这个就是JVM的Dump文件**。



![image-20200614162727737](/images/jvm-01/image-20200614162727737.png)

我们直接双击这个文件

![image-20200614163225649](/images/jvm-01/image-20200614163225649.png)

![image-20200614163300251](/images/jvm-01/image-20200614163300251.png)

这里就可以 定位到出现问题的地方。

# 11、垃圾回收机制（GC）

![image-20200614163519871](/images/jvm-01/image-20200614163519871.png)

垃圾回收主要是处理 **新生区、幸存区、养老区**的数据。

## 11.1 GC两种分类

- 轻GC(普通GC)
- 重GC（全局）

## 11.2 GC的常用算法

- 复制算法
- 标记清除算法
- 标记压缩算法

# 12、JMM （Java  Memory Model）java内存模型

作用：缓存一致性协议，用于定义数据读写规则。







