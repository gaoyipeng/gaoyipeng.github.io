---
title: Spring Boot整合Dubbo
date: 2020-10-30 15:16:18
tags: [Dubbo]
categories: [Dubbo]
description: Spring Boot整合Dubbo
typora-root-url: ..
---

Dubbo是阿里巴巴公司开源的一个高性能优秀的服务框架，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和Spring框架无缝集成。

本节学习SpringBoot集成Dubbo。Dubbo 的官网地址：http://dubbo.apache.org/zh-cn/

duubo的架构图：

![image-20201101211901922](/images/dubbo-01/image-20201101211901922.png)



## 1、注册中心

在进行Dubbo前我们需要准备一个注册中心。就是架构图中的Registry。Dubbo支持的注册中心很多，但是官方推荐使用Zookeeper.

![image-20201101204950403](/images/dubbo-01/image-20201101204950403.png)

此处我们下载一个Zookeeper。地址：https://zookeeper.apache.org/

![image-20201101205540238](/images/dubbo-01/image-20201101205540238.png)

当前最新版本是3.6.2：

![image-20201101205803361](/images/dubbo-01/image-20201101205803361.png)

下载后解压，将config目录下的zoo_sample.cfg重命名为zoo.cfg(Zookeeper配置文件，默认端口为2181，

另外还需修改临时数据保存目录，在config同目录下新建一个data文件夹。zoo.cfg修改内容如下：

![image-20201101211025261](/images/dubbo-01/image-20201101211025261.png)

然后双击bin目录下的zkServer.cmd启动即可。

![image-20201101211717925](/images/dubbo-01/image-20201101211717925.png)

## 2、监控中心

Dubbo给我们提供了dubbo-admin和dubbo-monitor-simple用于监控Dubbo服务，提供可视化界面来监控接口暴露，服务注册情况，也可以显示接口的调用明细和调用时间。下载地址：https://github.com/apache/dubbo-admin/tree/master



## 3、聚合工程

![image-20201030154213254](/images/dubbo-01/image-20201030154213254.png)

![image-20201030154241644](/images/dubbo-01/image-20201030154241644.png)

删除src目录，修改POM如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.sxdx</groupId>
    <artifactId>dubbo-boot</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>server-provider</module>
        <module>common-api</module>
        <module>service-consumer</module>
    </modules>


    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <dubbo.spring.boot.starter.version>2.7.7</dubbo.spring.boot.starter.version>
        <lombok.version>1.18.10</lombok.version>
        <zookeeper.version>3.6.2</zookeeper.version>
        <curator.version>4.0.1</curator.version>
        <spring-boot.version>2.1.6.RELEASE</spring-boot.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-spring-boot-starter</artifactId>
                <version>${dubbo.spring.boot.starter.version}</version>
            </dependency>

            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-framework</artifactId>
                <version>${curator.version}</version>
            </dependency>

            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-recipes</artifactId>
                <version>${curator.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </dependency>

            <dependency>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
                <version>${zookeeper.version}</version>
                <exclusions>
                    <exclusion>
                        <groupId>org.slf4j</groupId>
                        <artifactId>slf4j-log4j12</artifactId>
                    </exclusion>
                    <exclusion>
                        <groupId>log4j</groupId>
                        <artifactId>log4j</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>

            <!--<dependency>
                <groupId>com.101tec</groupId>
                <artifactId>zkclient</artifactId>
                <version>0.9</version>
                <exclusions>
                    <exclusion>
                        <groupId>org.slf4j</groupId>
                        <artifactId>slf4j-log4j12</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>-->
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

### 3.1 构建common-api 公共服务

新建一个module,common-api,保存公用Service接口。

pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbo-boot</artifactId>
        <groupId>com.sxdx</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>common-api</artifactId>


</project>
```

提供一个Service接口：

```java
package com.sxdx.common.api.service;

public interface HelloService {
    String hello(String str);
}

```

### 3.1 构建server-provider 服务提供者

新建一个服务提供方,用于暴露Dubbo服务

![image-20201030155040648](/images/dubbo-01/image-20201030155040648.png)

![image-20201030155225752](/images/dubbo-01/image-20201030155225752.png)

POM文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbo-boot</artifactId>
        <groupId>com.sxdx</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>server-provider</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.sxdx</groupId>
            <artifactId>common-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </dependency>

 <!--       <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.9</version>
        </dependency>
-->

    </dependencies>

</project>
```

这里我们引入了common-api模块。新建启动类(在Spring Boot启动类中我们加入`@EnableDubbo`注解，表示要开启dubbo功能)

```java
package com.sxdx.service.provider;

import org.apache.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@EnableDubbo
@SpringBootApplication
public class ServiceProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceProviderApplication.class, args);
        System.out.println("ServiceProvider-启动成功");
    }
}

```

新建application.yml，配置Dubbo：

```yaml
server:
  port: 1111

dubbo:
  application:
    # 服务名称，保持唯一
    name: server-provider
    # zookeeper地址，用于向其注册服务
  registry:
    address: zookeeper://127.0.0.1:2181
  #暴露服务方式
  protocol:
    # dubbo协议，固定写法
    name: dubbo
    # 暴露服务端口 （默认是20880，不同的服务提供者端口不能重复）
    port: 20880
  monitor:
    protocol: registry
```

如果Zookeeper是集群的话，`spring.dubbo.registry.address`配置为：

```
spring:
  dubbo:
    registry:
      address: zookeeper://127.0.0.1:2181?backup=127.0.0.1:2180,127.0.0.1:2182
```

接下来我们在`com.sxdx.service.provider.service`路径下创建一个`HelloService`接口的实现类：

```java
package com.sxdx.service.provider.service;

import com.alibaba.dubbo.config.annotation.Service;
import com.sxdx.common.api.service.HelloService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Slf4j
@Service
@Component
public class HelloServiceImpl implements HelloService {
    @Override
    public String hello(String str) {
        log.info("dubbo服务调用");
        return "hello," + str;
    }
}

```

值得注意的是`@Service`注解为Dubbo提供的`com.alibaba.dubbo.config.annotation.Service`，而非Spring的那个。

### 3.2 构建service-consumer 服务消费者

![image-20201109143142392](/images/dubbo-01/image-20201109143142392.png)

POM文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbo-boot</artifactId>
        <groupId>com.sxdx</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>service-consumer</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.sxdx</groupId>
            <artifactId>common-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </dependency>

 <!--       <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.9</version>
        </dependency>-->

    </dependencies>

</project>
```

这里我们同样引入了common-api模块，我们新建springboot启动类如下：

```java
package com.sxdx.service.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @program: dubbo-boot
 * @description:
 * @author: garnett
 * @create: 2020-11-05 15:17
 **/

@SpringBootApplication
public class ServiceConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceConsumerApplication.class, args);
        System.out.println("ServiceConsumer-启动成功");
    }
}
```

yam配置如下：

```yaml
server:
  port: 2222

dubbo:
  application:
    # 服务名称，保持唯一
    name: server-consumer
    # zookeeper地址，用于从中获取注册的服务
  registry:
    address: zookeeper://127.0.0.1:2181
  protocol:
    # dubbo协议，固定写法
    name: dubbo
  monitor:
    protocol: registry
```

我们写一个方法来消费dubbo服务：

```java
package com.sxdx.service.consumer.controller;

import com.alibaba.dubbo.config.annotation.Reference;
import com.sxdx.common.api.service.HelloService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @program: dubbo-boot
 * @description:
 * @author: garnett
 * @create: 2020-11-05 15:04
 **/
@RestController
public class HelloController {

    @Reference(check = false)
    private HelloService helloService;

    @GetMapping("/hello")
    public String hello(){
        return this.helloService.hello("Garnett");
    }
}

```

## 4、验证

我们启动Zookeeper服务、service-provider和service-consumer：

![image-20201109150023279](/images/dubbo-01/image-20201109150023279.png)

![image-20201109150007645](/images/dubbo-01/image-20201109150007645.png)

可以看到调用成功了。

