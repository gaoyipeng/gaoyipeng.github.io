---
title: Activiti（27） 搭建认证服务器
date: 2020-09-28 16:59:32
tags: [Activiti,Spring-Security-OAuth2]
categories: [Activiti,Spring-Security-OAuth2]
description: Activiti（27） 搭建认证服务器
typora-root-url: ..
---

## 1、搭建认证服务器

我们希望搭建的各个微服务系统是受保护的，只有通过合法的认证信息才能访问相关资源。我们搭建一个统一给微服务发放访问令牌的认证服务器：workflow-auth。

我们新建一个module:

![image-20200928145259651](/images/activiti6-27/image-20200928145259651.png)

![image-20200928145359591](/images/activiti6-27/image-20200928145359591.png)

修改workflow-auth的pom，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>workflow-activiti</artifactId>
        <groupId>com.sxdx</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>workflow-auth</artifactId>
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-security</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

在main-java目录下补全springboot启动文件：

```java
package com.sxdx.workflow.auth;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class WorkFlowAuthApplication {

    public static void main(String[] args) {
        SpringApplication.run(WorkFlowAuthApplication.class, args);
    }
}
```

编写配置文件application.yml，内容如下所示：

```xml
spring:
  application:
    name: workflow-auth
  thymeleaf:
    cache: false
    prefix: classpath:/templates/
    suffix: .html
    mode: HTML5
    encoding: UTF-8
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8

server:
  port: 8101
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

```

另外需要集成logBack日志框架（前面章节已经介绍过了，不再赘述），在resources下新建`logback-spring.xml`，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">

    <springProperty scope="context" name="springAppName" source="spring.application.name"/>

    <property name="log.path" value="logs/workflow-auth" />
    <property name="log.maxHistory" value="15"/>
    <property name="log.colorPattern" value="%magenta(%d{yyyy-MM-dd HH:mm:ss}) %highlight(%-5level) %boldCyan(${springAppName:-}) %yellow(%thread) %green(%logger) %msg%n"/>
    <property name="log.pattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5level ${springAppName:-} %thread %logger %msg%n"/>

    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${log.colorPattern}</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>info</level>
        </filter>
    </appender>

    <!--输出到debug-->
    <appender name="debug" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/debug/logback-debug-%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <!--编码-->
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印DEBUG日志 -->
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--输出到info-->
    <appender name="info" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/info/logback-info-%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印INFO日志 -->
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--输出到warn-->
    <appender name="warn" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/warn/logback-warn-%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印WARN日志 -->
            <level>WARN</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--输出到error-->
    <appender name="error" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/error/logback-error-%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印ERROR日志 -->
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--不同环境分别设置对应的日志输出节点 -->
    <!--开发-->
    <springProfile name="dev">
        <root level="debug">
            <appender-ref ref="console" />
            <appender-ref ref="debug" />
            <appender-ref ref="info" />
            <appender-ref ref="warn" />
            <appender-ref ref="error" />
        </root>
    </springProfile>

    <!--生产环境-->
    <springProfile name="release">
        <root level="info">
            <appender-ref ref="console" />
            <appender-ref ref="debug" />
            <appender-ref ref="info" />
            <appender-ref ref="warn" />
            <appender-ref ref="error" />
        </root>
    </springProfile>

</configuration>
```

### 2.1  创建认证服务类

认证服务器需要创建**三大配置**：

- 认证服务配置：负责发放、校验令牌
- 资源服务配置：这一项是可选的，因为认证服务器同样也可以是一个资源服务器。
- WebSecurity配置：通用配置

我们先创建认证服务类 AuthorizationServerConfigurer：

```java
package com.sxdx.workflow.auth.config;

import com.sxdx.workflow.auth.service.OauthUserDetailService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.ClientDetailsService;
import org.springframework.security.oauth2.provider.code.AuthorizationCodeServices;
import org.springframework.security.oauth2.provider.code.InMemoryAuthorizationCodeServices;
import org.springframework.security.oauth2.provider.token.DefaultTokenServices;
import org.springframework.security.oauth2.provider.token.TokenStore;

/**
 * 认证服务器配置
 */
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfigurer extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private PasswordEncoder passwordEncoder;
    @Autowired
    private TokenStore tokenStore;
    @Autowired
    private ClientDetailsService clientDetailsService;
    @Autowired
    private AuthorizationCodeServices authorizationCodeServices;
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private OauthUserDetailService userDetailService;


    /**
     * 配置客户端详情信息
     * @param clients
     * @throws Exception
     */
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("kiki")
                .secret(passwordEncoder.encode("123456"))
                .authorizedGrantTypes("password","authorization_code","client_credentials" ,"implicit","refresh_token")
                .autoApprove(true)
                .scopes("all")
                .redirectUris("https://www.baidu.com")
                .resourceIds("activiti-rest","activiti-web","activiti-auth");
    }

    /**
     * 配置令牌端点的安全约束
     * @param security
     * @throws Exception
     */
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security
                .tokenKeyAccess("permitAll()")
                .checkTokenAccess("permitAll()")
                .allowFormAuthenticationForClients();//允许表单认证
    }

    /**ClientCredentialsTokenEndpointFilter
     * 配置令牌（token）的访问端点和令牌服务（token service）
     * @param endpoints
     * @throws Exception
     */
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                .authenticationManager(authenticationManager)//认证管理器
                .authorizationCodeServices(authorizationCodeServices)//授权码服务
                .tokenServices(tokenServices())//令牌管理服务
                .userDetailsService(userDetailService)
                .allowedTokenEndpointRequestMethods(HttpMethod.POST);
    }

    /**
     * 令牌管理服务
     * @return
     */
    @Primary
    @Bean
    public DefaultTokenServices tokenServices(){
        DefaultTokenServices tokenServices = new DefaultTokenServices();
        tokenServices.setClientDetailsService(clientDetailsService);//客户端详情服务
        tokenServices.setTokenStore(tokenStore);//token存储策略
        tokenServices.setSupportRefreshToken(true);//支持token刷新
        tokenServices.setAccessTokenValiditySeconds(60 * 60 * 24);//令牌默认有效期
        tokenServices.setRefreshTokenValiditySeconds(60 * 60 * 24 * 7);//刷新令牌默认有效期
        return tokenServices;
    }

    /**
     * 授权码服务：配置授权码模式下授权码存储方式
     * @return
     */
    @Bean
    public AuthorizationCodeServices authorizationCodeServices(){
        return new InMemoryAuthorizationCodeServices();
    }

}

```

我们来介绍下认证服务类，首先`@EnableAuthorizationServer`表示开始认证服务。而且继承了`AuthorizationServerConfigurerAdapter`，`AuthorizationServerConfigurerAdapter`包含了3个方法：

```java
public class AuthorizationServerConfigurerAdapter implements AuthorizationServerConfigurer {
    public AuthorizationServerConfigurerAdapter() {
    }

    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
    }

    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    }

    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    }
}
```

分别用于配置 **令牌端点的安全约束、客户端信息、配置令牌端点的安全约束**

这也是我们`AuthorizationServerConfigurer`类的骨架，其他代码都是围绕这3个方法产生的，都有注解说明。

其中我们介绍下passwordEncoder、