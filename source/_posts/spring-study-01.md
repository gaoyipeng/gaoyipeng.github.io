---
title: Spring源码学习（一）：环境准备
date: 2020-03-14 16:03:51
tags: Spring
categories: Spring
description: 本地IDEA导入spring-framework-5.1.0.RC1.zip
---

## 构建工具 Gradle

Spring 源码不是使用我们常用的Maven构建，使用Gradle。 Gradle不是我们介绍的重点，我们在本地Window10环境下配置好环境即可。

参考：[Gradle的安装与配置](https://www.cnblogs.com/NyanKoSenSei/p/11458953.html)

![gradle-v.png](/images/spring/gradle-v.png)

## Spring 源码包准备

### 下载预编译

从Github下载源码包：[下载地址](https://github.com/spring-projects/spring-framework/archive/v5.1.0.RC1.zip),我下载的是：spring-framework-5.1.0.RC1.zip

解压文件，查看import-into-idea.md文件：
````
The following has been tested against IntelliJ IDEA 2016.2.2

## Steps

_Within your locally cloned spring-framework working directory:_

1. Precompile `spring-oxm` with `./gradlew :spring-oxm:compileTestJava`
2. Import into IntelliJ (File -> New -> Project from Existing Sources -> Navigate to directory -> Select build.gradle)
3. When prompted exclude the `spring-aspects` module (or after the import via File-> Project Structure -> Modules)
4. Code away

## Known issues

1. `spring-core` and `spring-oxm` should be pre-compiled due to repackaged dependencies.
See `*RepackJar` tasks in the build and https://youtrack.jetbrains.com/issue/IDEA-160605).
2. `spring-aspects` does not compile due to references to aspect types unknown to
IntelliJ IDEA. See https://youtrack.jetbrains.com/issue/IDEA-64446 for details. In the meantime, the
'spring-aspects' can be excluded from the project to avoid compilation errors.
3. While JUnit tests pass from the command line with Gradle, some may fail when run from
IntelliJ IDEA. Resolving this is a work in progress. If attempting to run all JUnit tests from within
IntelliJ IDEA, you will likely need to set the following VM options to avoid out of memory errors:
    -XX:MaxPermSize=2048m -Xmx2048m -XX:MaxHeapSize=2048m
4. If you invoke "Rebuild Project" in the IDE, you'll have to generate some test
resources of the `spring-oxm` module again (`./gradlew :spring-oxm:compileTestJava`)    


## Tips

In any case, please do not check in your own generated .iml, .ipr, or .iws files.
You'll notice these files are already intentionally in .gitignore. The same policy goes for eclipse metadata.

## FAQ

Q. What about IntelliJ IDEA's own [Gradle support](https://confluence.jetbrains.net/display/IDEADEV/Gradle+integration)?

A. Keep an eye on https://youtrack.jetbrains.com/issue/IDEA-53476

````
在解压目录下执行预编译命令：gradlew :spring-oxm:compileTestJava命令

![init-project.png](/images/spring/init-project.png)
![init-project1.png](/images/spring/init-project1.png)


### 导入IDEA

import-into-idea.md文件，已经说明了导入步骤：

File -> New -> Project from Existing Sources -> Navigate to directory -> Select build.gradle

然后耐心等待jar包下载完 ....

![spring-idea.png](/images/spring/spring-idea.png)


### 运行测试

打开SimplePropertyNamespaceHandlerWithExpressionLanguageTests.java，执行单元测试。
![init-test.png](/images/spring/init-test.png)

到这一步，基本完工，本节的目的只是为了导入spring工程后代码别飘红就行，要不看着报错很糟心。


接下来开始源码学习，希望自己可以坚持下去，不写了，打把LOL去。
