---
title: Maven聚合工程-多环境打包配置
date: 2021-02-04 17:10:09
tags: Maven
categories: Maven
description:  Maven聚合工程-多环境打包配置
---

## 1、问题场景

分布式微服务目前已经非常行，一个工程中经常会包含几个或者十几个独立的服务，我们经常搭建Maven聚合工程，子工程是一个个的`SpringBoot`应用。工程往往有测试环境(test)、开发环境(dev)、生产环境(prod)等.

每个环境的配置是不同的，所以我们使用`application-xx.yaml`来分别配置各个环境参数，使用`spring.profiles.active`来激活对应的环境，但是打包时如果手动去修改各个子工程的`spring.profiles.active`会很麻烦，而且打出的包中包含所有环境的配置变量，不是很安全，这个如何处理呢？

先贴出示例工程：

![image-20210204172257678](D:\hexo\gaoyipeng.github.io\source\images\maven-01\image-20210204172257678.png)

`sms-platform`是一个聚合工程，其中`sms-nafmii`、`sms-provider`是两个`springboot`应用，都分别有4个环境（`dev`、`moni`、`prod`、`officeprod`）,切换环境只需修改`application.yaml`如下代码：

```
spring:
  profiles:
    active: dev
```

此时如果我们想要部署prod环境，我们将`sms-nafmii`、`sms-provider`工程的`application.yaml`的`spring.profiles.active`都改为prod，如果只改2个还好，如果有10几个工程呢？

## 2、解决方法

1、在父工程的`pom.xml`中添加如下内容：

```yaml
   <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>${project.build.sourceEncoding}</encoding>
                </configuration>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <!-- 处理文件时替换文件中的变量 -->
                <filtering>true</filtering>
                <excludes>
                    <!-- 打包时排除文件，可自行添加test.yml -->
                    <exclude>application.yaml</exclude>
                    <exclude>application-dev.yaml</exclude>
                    <exclude>application-moni.yaml</exclude>
                    <exclude>application-officeprod.yaml</exclude>
                    <exclude>application-prod.yaml</exclude>
                </excludes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
                <!-- 打包时所包含得文件 -->
                <includes>
                    <include>application.yaml</include>
                    <include>application-${profileActive}.yaml</include>
                </includes>
            </resource>
        </resources>

    </build>
    <profiles>
        <!-- dev开发环境配置,prod为生产环境配置 -->
        <profile>
            <id>dev</id>
            <properties>
                <profileActive>dev</profileActive>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <id>moni</id>
            <properties>
                <profileActive>moni</profileActive>
            </properties>
        </profile>
        <profile>
            <id>officeprod</id>
            <properties>
                <profileActive>officeprod</profileActive>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <profileActive>prod</profileActive>
            </properties>
        </profile>
    </profiles>

```

2、修改各个子工程的`application.yaml`

![image-20210204173903899](D:\hexo\gaoyipeng.github.io\source\images\maven-01\image-20210204173903899.png)

3、打包

![image-20210204174009314](D:\hexo\gaoyipeng.github.io\source\images\maven-01\image-20210204174009314.png)



![image-20210204174214571](D:\hexo\gaoyipeng.github.io\source\images\maven-01\image-20210204174214571.png)

打出包中只包含公共配置文件和对应环境的文件。

完工！！！！