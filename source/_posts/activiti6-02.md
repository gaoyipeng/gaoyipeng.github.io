---
title: Activiti（2）-环境搭建
date: 2020-05-28 10:10:57
tags: Activiti
categories: Activiti
description: Activiti（2）-环境搭建
typora-root-url: ..
---

> 本节我们使用springboot集成Activiti 环境。

环境说明：

- IDEA: 2019.3
- JDK1.8
- Maven: 3.6.3
- SpringBoot:2.1.1
- Activiti :6.0.0
- Mysql :5.7

# 1、搭建maven聚合工程：

> 为了以后扩展方便，再此我们搭建一个聚合工程。

## 1.1 框架搭建

![image-20200528103143771](/images/activiti/activiti6-02/image-20200528103143771.png)



![image-20200528103347641](/images/activiti/activiti6-02/image-20200528103347641.png)

删除src目录

![image-20200528103428378](/images/activiti/activiti6-02/image-20200528103428378.png)

创建一个module工程

![image-20200528103452128](/images/activiti/activiti6-02/image-20200528103452128.png)

![image-20200528103557655](/images/activiti/activiti6-02/image-20200528103557655.png)

目录结构如下：

![image-20200528104038882](/images/activiti/activiti6-02/image-20200528104038882.png)



## 1.2 pom文件

- workflow-activiti父工程pom文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>org.sxdx</groupId>
      <artifactId>workflow-activiti</artifactId>
      <packaging>pom</packaging>
      <version>1.0-SNAPSHOT</version>
      <modules>
          <module>workflow-activiti-rest</module>
      </modules>
      <properties>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
          <java.version>1.8</java.version>
          <druid.version>1.1.14</druid.version>
          <swagger.version>2.9.2</swagger.version>
          <fastjson.version>1.2.68</fastjson.version>
          <activiti.version>6.0.0</activiti.version>
      </properties>
  
      <!-- 依赖声明 -->
      <dependencyManagement>
          <dependencies>
              <!-- SpringBoot的依赖配置-->
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-dependencies</artifactId>
                  <version>2.1.1.RELEASE</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
              <!--阿里数据库连接池 -->
              <dependency>
                  <groupId>com.alibaba</groupId>
                  <artifactId>druid-spring-boot-starter</artifactId>
                  <version>${druid.version}</version>
              </dependency>
              <!-- swagger2-->
              <dependency>
                  <groupId>io.springfox</groupId>
                  <artifactId>springfox-swagger2</artifactId>
                  <version>${swagger.version}</version>
                  <exclusions>
                      <exclusion>
                          <groupId>io.swagger</groupId>
                          <artifactId>swagger-annotations</artifactId>
                      </exclusion>
                      <exclusion>
                          <groupId>io.swagger</groupId>
                          <artifactId>swagger-models</artifactId>
                      </exclusion>
                  </exclusions>
              </dependency>
  
              <!-- swagger2-UI-->
              <dependency>
                  <groupId>io.springfox</groupId>
                  <artifactId>springfox-swagger-ui</artifactId>
                  <version>${swagger.version}</version>
              </dependency>
  
              <!-- 阿里JSON解析器 -->
              <dependency>
                  <groupId>com.alibaba</groupId>
                  <artifactId>fastjson</artifactId>
                  <version>${fastjson.version}</version>
              </dependency>
  
          </dependencies>
      </dependencyManagement>
  
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
      </build>
  
  </project>
  ```



- workflow-activiti-rest 子工程

  ```xml
   <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <parent>
          <artifactId>workflow-activiti</artifactId>
          <groupId>org.sxdx</groupId>
          <version>1.0-SNAPSHOT</version>
      </parent>
      <modelVersion>4.0.0</modelVersion>
  
      <artifactId>workflow-activiti-rest</artifactId>
  
      <dependencies>
  
          <!-- SpringBoot Web容器 -->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
  
          <dependency>
              <groupId>org.mybatis.spring.boot</groupId>
              <artifactId>mybatis-spring-boot-starter</artifactId>
              <version>1.3.2</version>
          </dependency>
  
  
          <!-- Mysql驱动包 -->
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
          </dependency>
  
          <!--阿里数据库连接池 -->
          <dependency>
              <groupId>com.alibaba</groupId>
              <artifactId>druid-spring-boot-starter</artifactId>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
          </dependency>
  
          <!-- spring-boot-devtools -->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-devtools</artifactId>
              <optional>true</optional> <!-- 表示依赖不会传递 -->
          </dependency>
  
      </dependencies>
  
      <build>
          <plugins>
              <plugin>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-maven-plugin</artifactId>
                  <version>2.1.1.RELEASE</version>
                  <configuration>
                      <fork>true</fork> <!-- 如果没有该配置，devtools不会生效 -->
                  </configuration>
                  <executions>
                      <execution>
                          <goals>
                              <goal>repackage</goal>
                          </goals>
                      </execution>
                  </executions>
              </plugin>
          </plugins>
          <finalName>${project.artifactId}</finalName>
      </build>
  
  </project>
  ```
  
  

# 2、改成springboot工程

## 2.1  设置启动类

```java
package com.sxdx.workflow.activiti.rest;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class WorkFlowActivitiRestApplication {
    public static void main(String[] args)
    {
        SpringApplication.run(WorkFlowActivitiRestApplication.class, args);
        System.out.println("启动成功");
    }
}

```

## 2.2 application.yml

```yaml
# 开发环境配置
server:
  # 服务器的HTTP端口，默认为80
  port: 8080
  servlet:
    # 应用的访问路径
    context-path: /
  tomcat:
    # tomcat的URI编码
    uri-encoding: UTF-8
    # tomcat最大线程数，默认为200
    max-threads: 800
    # Tomcat启动初始化的线程数，默认值25
    min-spare-threads: 30

# Spring配置
spring:
  datasource:
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/activiti?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true
    username: root
    password: gaoyipeng
    #druid配置
    druid:
      # 初始连接数
      initialSize: 5
      # 最小连接池数量
      minIdle: 10
      # 最大连接池数量
      maxActive: 20
      # 配置获取连接等待超时的时间
      maxWait: 60000
      # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
      timeBetweenEvictionRunsMillis: 60000
      # 配置一个连接在池中最小生存的时间，单位是毫秒
      minEvictableIdleTimeMillis: 300000
      # 配置一个连接在池中最大生存的时间，单位是毫秒
      maxEvictableIdleTimeMillis: 900000
      # 配置检测连接是否有效
      validationQuery: SELECT 1 FROM DUAL
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      webStatFilter:
        enabled: true
      statViewServlet:
        enabled: true
        # 设置白名单，不填则允许所有访问
        allow:
        url-pattern: /druid/*
        # 控制台管理用户名和密码
        login-username:
        login-password:
      filter:
        stat:
          enabled: true
          # 慢SQL记录
          log-slow-sql: true
          slow-sql-millis: 1000
          merge-sql: true
        wall:
          config:
            multi-statement-allow: true
```

通过我们上面的配置我们知道需要新建一个名为activiti的mysql数据库。

最终效果如下：

![image-20200528105814806](/images/activiti/activiti6-02/image-20200528105814806.png)

# 3、启动

访问：http://localhost:8080/druid

![image-20200528110858638](/images/activiti/activiti6-02/image-20200528110858638.png)

# 4、集成activiti6

在workflow-activiti-rest 的pom 文件中添加

```xml
     <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-spring-boot-starter-rest-api</artifactId>
            <version>${activiti.version}</version>
        </dependency>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-json-converter</artifactId>
            <version>6.0.0</version>
            <exclusions>
                <exclusion>
                    <groupId>org.activiti</groupId>
                    <artifactId>activiti-bpmn-model</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- xml解析依赖-->
        <dependency>
            <groupId>org.apache.xmlgraphics</groupId>
            <artifactId>batik-codec</artifactId>
            <version>1.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.xmlgraphics</groupId>
            <artifactId>batik-css</artifactId>
            <version> 1.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.xmlgraphics</groupId>
            <artifactId>batik-svg-dom</artifactId>
            <version>1.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.xmlgraphics</groupId>
            <artifactId>batik-svggen</artifactId>
            <version>1.7</version>
        </dependency>
```

## 4.1 排除 Activiti 和 Spring Security 登录验证

再次启动会报错：

```
org.springframework.beans.factory.BeanDefinitionStoreException: Failed to process import candidates for configuration class [com.sxdx.workflow.activiti.rest.WorkFlowActivitiRestApplication]; nested exception is java.io.FileNotFoundException: class path resource [org/springframework/security/config/annotation/authentication/configurers/GlobalAuthenticationConfigurerAdapter.class] cannot be opened because it does not exist
......

Caused by: java.io.FileNotFoundException: class path resource [org/springframework/security/config/annotation/authentication/configurers/GlobalAuthenticationConfigurerAdapter.class] cannot be opened because it does not exist
	at org.springframework.core.io.ClassPathResource.getInputStream(ClassPathResource.java:180) ~[spring-core-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.core.type.classreading.SimpleMetadataReader.<init>(SimpleMetadataReader.java:51) ~[spring-core-5.1.3.RELEASE.jar:5.1.3.RELEASE]
```

解决方法：

```java 
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class,
        org.activiti.spring.boot.SecurityAutoConfiguration.class,
        org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration.class
})
public class WorkFlowActivitiRestApplication {
    public static void main(String[] args)
    {
        SpringApplication.run(WorkFlowActivitiRestApplication.class, args);
        System.out.println("启动成功");
    }
}
```

## 4.2  application.yml 添加配置

```yaml
# Spring配置
spring:
  # activiti 模块
  # 解决启动报错：class path resource [processes/] cannot be resolved to URL because it does not exist
  activiti:
    check-process-definitions: false
```



![image-20200528112705109](/images/activiti/activiti6-02/image-20200528112705109.png)

# 5、验证

再次启动程序，能够正常启动。

![image-20200528112935881](/images/activiti/activiti6-02/image-20200528112935881.png)

这是我们看看数据库：

![image-20200528113032536](/images/activiti/activiti6-02/image-20200528113032536.png)

可以看到自动生成了activiti的28张表。至此，环境已经搭建完成。