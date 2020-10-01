---
title: Activiti（27） 搭建认证服务器
date: 2020-09-28 16:59:32
tags: [Activiti,Spring-Security-OAuth2]
categories: [Activiti,Spring-Security-OAuth2]
description: Activiti（27） 搭建认证服务器
typora-root-url: ..
---

## 1、搭建认证服务器

我们希望搭建的各个微服务系统是受保护的，只有通过合法的认证信息才能访问相关资源。我们搭建一个统一给微服务发放访问令牌的认证服务器：**workflow-auth**。

我们新建一个module:

![image-20200928145259651](/images/activiti6-27/image-20200928145259651.png)

![image-20200928145359591](/images/activiti6-27/image-20200928145359591.png)

修改`workflow-auth`的`pom`，内容如下：

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
        <dependency>
            <artifactId>workflow-common</artifactId>
            <groupId>com.sxdx</groupId>
            <version>1.0-SNAPSHOT</version>
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

在`main/java`目录下补全`SpringBoot`启动文件：

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

编写配置文件`application.yml`，内容如下所示：

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

另外需要集成`logBack`日志框架（前面章节已经介绍过了，不再赘述），在resources下新建`logback-spring.xml`，内容如下：

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

### 1.1  创建认证服务类

认证服务器需要创建**三大配置**：

- 认证服务配置：负责发放、校验令牌
- 资源服务配置：这一项是可选的，因为认证服务器同样也可以是一个资源服务器。
- WebSecurity配置：Security配置，主要处理除资源服务外的其他服务请求。

我们先创建认证服务类 `AuthorizationServerConfig`：

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
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

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

我们来介绍下认证服务类，首先`@EnableAuthorizationServer`表示开始认证服务。而且继承了`AuthorizationServerConfigurerAdapter`适配器，`AuthorizationServerConfigurerAdapter`包含了3个方法：

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

这也是我们`AuthorizationServerConfig`类的骨架，其他代码都是围绕这3个方法产生的，都有注解说明。

接下来我们分别介绍下以下几个Bean：

- passwordEncoder：定义加密算法
- tokenStore：定义token存储方式
- clientDetailsService：定义客户端client信息
- authorizationCodeServices：`oauth2`授权码模式需要的服务，此处定义授权码模式code保存在内存中。
- authenticationManager：`oauth2`密码模式需要的服务，这个我们在下面的`WebSecurity`配置中添加。
- userDetailService：校验用户名密码的类（从数据库或内存中校验用户信息）

### 1.2 创建资源服务配置类

虽然我们现在正在搭建的`workflow-auth`是一个认证服务器，但是认证服务器本身也可以对外提供REST服务，比如通过Token获取当前登录用户信息，注销当前Token等，所以它也是一台资源服务器。于是我们需要定义一个资源服务器的配置类

我们创建资源服务 `ResourceServerConfig`：

```java
package com.sxdx.workflow.auth.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;

/**
 * @program: workflow-auth
 * @description: 资源服务配置
 * @author: garnett
 * @create: 2020-09-26 17:05
 **/
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    public static final String RESOURCE_ID = "activiti-auth";

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.resourceId(RESOURCE_ID)//重点，设置资源id
                .stateless(true);
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .requestMatchers().antMatchers("/**")
                .and()
                .authorizeRequests()
                .antMatchers("/**").authenticated();
    }

}

```

`ResourceServerConfig`继承了`ResourceServerConfigurerAdapter`，并重写了`configure(HttpSecurity http)`方法和`configure(ResourceServerSecurityConfigurer resources)`方法，通过`requestMatchers().antMatchers("/**")`的配置表明该安全配置对所有请求都生效。

类上的`@EnableResourceServer`用于开启资源服务器相关配置。而`ResourceServerSecurityConfigurer`则定义了本服务的`resourceId`。

### 1.3 创建WebSecurity配置类

主要处理除资源服务外的其他其他服务请求。

```java
package com.sxdx.workflow.auth.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Order(2)
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfigure extends WebSecurityConfigurerAdapter {
    /**
     * Spring Security内部实现好的 BCryptPasswordEncoder。
     * BCryptPasswordEncoder的特点就是，对于一个相同的密码，每次加密出来的加密串都不同
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    /**
     * 认证管理器（密码模式需要用到）
     * @return
     * @throws Exception
     */
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .requestMatchers()
                .antMatchers("/oauth/**","/login/**","/logout/**")
                .and()
                .authorizeRequests()
                .antMatchers("/oauth/**").authenticated()
                .and()
                .formLogin().permitAll();
    }
}

```

该类继承了`WebSecurityConfigurerAdapter`适配器，重写了几个方法，并且使用`@EnableWebSecurity`注解标注，开启了和Web相关的安全配置。`@EnableGlobalMethodSecurity(prePostEnabled = true)`表示开启了security权限注解。

我们定义了一个`PasswordEncoder`类型的Bean，该类是一个接口，定义了几个和密码加密校验相关的方法，这里我们使用的是Spring Security内部实现好的`BCryptPasswordEncoder`。`BCryptPasswordEncoder`的特点就是，对于一个相同的密码，每次加密出来的加密串都不同：

```java
public static void main(String[] args) {
    String password = "123456";
    System.out.println(new BCryptPasswordEncoder().encode(password));
    System.out.println(new BCryptPasswordEncoder().encode(password));
}
```

输出结果：

```
$2a$10$TgKIGaJrL8LBFT8bEj8gH.3ctyo1PpSTw4fs4o6RuMOE4R665HdpS
$2a$10$ZEcCOMVVIV5SfoXPXih92uGJfVeaugMr/PydhYnLvsCroS9xWjOIq
```

然后我们定义了一个`AuthenticationManager`类型的Bean，这就是我们在认证服务配置类中注入的`authenticationManager`

最后还重写了`WebSecurityConfigurerAdapter`类的`configure(HttpSecurity http)`方法，其中`requestMatchers().antMatchers("/oauth/**","/login/**","/logout/**")`的含义是：`SecurityConfig`安全配置类只对`/oauth/``/login/``/logout/`开头的请求有效。

这里重点说明下这个`@Order(2)`的原理：

代码写到这里可以发现在`ResourceServerConfig`、`SecurityConfig`配置类中都定义了HttpSecurity规则。`SecurityConfig`对`/oauth/``/login/``/logout/`开头的请求有效，`ResourceServerConfig`这对所有请求有效。那么当一个请求进来时，到底哪个安全配置先生效呢？其实`Spring Security`是基于过滤器链来工作的，哪个优先级高，就先执行哪个。

那么`SecurityConfig`和`ResourceServerConfig`的优先级是多少？首先我们查看`SecurityConfig`继承的类`WebSecurityConfigurerAdapter`的源码：

```
@Order(100)
public abstract class WebSecurityConfigurerAdapter implements WebSecurityConfigurer<WebSecurity> {
   ......
}
```

可以看到类上使用了`@Order(100)`标注，说明其顺序是100。

再来看看`ResourceServerConfig`类上`@EnableResourceServer`注解源码：

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({ResourceServerConfiguration.class})
public @interface EnableResourceServer {
}
```

该注解引入了`ResourceServerConfiguration`配置类，查看`ResourceServerConfiguration`源码：

```
@Configuration
public class ResourceServerConfiguration extends WebSecurityConfigurerAdapter implements Ordered {
    private int order = 3;
    ......
}
```

所以`ResourceServerConfig`的顺序是3。在Spring中，数字越小，优先级越高，也就是说`ResourceServerConfig`的优先级要高于`SecurityConfig`，这也就意味着所有请求都会被`ResourceServerConfig`过滤器链处理，包括`/oauth/`开头的请求。这显然不是我们要的效果，我们原本是希望以`/oauth/``/login/``/logout/`开头的请求由`SecurityConfigure`过滤器链处理，剩下的其他请求由`ResourceServerConfig`过滤器链处理。

为了解决上面的问题，我们可以手动指定这两个类的优先级，让`SecurityConfigure`的优先级高于`ResourceServerConfig`。在`SecurityConfig`类上使用`Order(2)`注解标注即可。

总结下`SecurityConfig`和`ResourceServerConfig`的区别吧：

1. `SecurityConfig`用于处理`/oauth/``/login/``/logout/`开头的请求，OAuth2内部定义的获取令牌，刷新令牌的请求地址都是以`/oauth/``/login/``/logout/`开头的，也就是说`SecurityConfig`用于处理和令牌相关的请求；
2. `ResourceServerConfig`用于处理非`/oauth/``/login/``/logout/`开头的请求，其主要用于资源的保护，客户端只能通过OAuth2协议发放的令牌来从资源服务器中获取受保护的资源。

### 1.4 创建 tokenStore

在认证服务类中，注入过一个TokenStore类型的Bean。我们专门新建一个类TokenConfig来配置token相关的信息，其中''InMemoryTokenStore''表示生成的token存储在内存中。

```java
package com.sxdx.workflow.auth.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.InMemoryTokenStore;

@Configuration
public class TokenConfig {
    @Bean
    public TokenStore tokenStore() {
        return new InMemoryTokenStore();
    }
}
```

### 1.5  创建 userDetailService

还需要定义一个用于校验用户名密码的类，也就是上面认证服务类中提到的`userDetailService`。代码如下所示：

```java
package com.sxdx.workflow.auth.service;

import com.sxdx.workflow.auth.entity.SecurityUser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

@Service
public class OauthUserDetailService implements UserDetailsService {
    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        SecurityUser user = new SecurityUser();
        user.setUsername(username);
        user.setPassword(this.passwordEncoder.encode("123456"));

        return new User(username, user.getPassword(), user.isEnabled(),
                user.isAccountNonExpired(), user.isCredentialsNonExpired(),
                user.isAccountNonLocked(), AuthorityUtils.commaSeparatedStringToAuthorityList("user:add"));
    }
}
```

`OauthUserDetailService`实现了`UserDetailsService`接口的`loadUserByUsername`方法。定义登录逻辑：用户名任意，密码为123456 即可通过`spring-security`登录。`loadUserByUsername`方法返回一个`UserDetails`对象，该对象也是一个接口，包含一些用于描述用户信息的方法，源码如下：

```$xslt
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    String getPassword();
    String getUsername();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

这些字段的含义如下：

* `getAuthorities`获取用户包含的权限，返回权限集合，权限是一个继承了`GrantedAuthority`的对象；
* `getPassword`和`getUsername`用于获取密码和用户名；
* `isAccountNonExpired`方法返回boolean类型，用于判断账户是否未过期，未过期返回true反之返回false；
* `isAccountNonLocked`方法用于判断账户是否未锁定；
* `isCredentialsNonExpired`用于判断用户凭证是否没过期，即密码是否未过期；
* `isEnabled`方法用于判断用户是否可用。

实际中我们可以自定义`UserDetails`接口的实现类，也可以直接使用Spring Security提供的`UserDetails`接口实现类`org.springframework.security.core.userdetails.User`。

`OauthUserDetailService`中`SecurityUser`为我们自定义的用户实体类，代表我们从数据库中查询出来的用户。代码如下：

```java
package com.sxdx.workflow.auth.entity;
import lombok.Data;
import org.springframework.security.core.GrantedAuthority;
import java.io.Serializable;
import java.util.Set;

@Data
public class SecurityUser implements Serializable {

    private static final long serialVersionUID = 3191927289420949930L;
    private String password;
    private String username;
    private Set<GrantedAuthority> authorities;
    private boolean accountNonExpired = true;
    private boolean accountNonLocked = true;
    private boolean credentialsNonExpired = true;
    private boolean enabled = true;
}
```

### 1.6 测试接口

此处我们写2个接口，用于后面的测试。

```java
package com.sxdx.workflow.auth.controller;

import com.sxdx.common.exception.base.CommonException;
import com.sxdx.common.util.CommonResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.Authentication;
import org.springframework.security.oauth2.provider.token.ConsumerTokenServices;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;

@RestController
public class AuthorizationController {

    @Autowired
    private ConsumerTokenServices consumerTokenServices;

    /**
     * 获取当前用户信息
     * @param authentication
     * @return
     */
    @GetMapping("/authentication")
    public Object authentication(Authentication authentication){
        return authentication;
    }

    /**
     * 退出登录
     * @param request
     * @return
     * @throws CommonException
     */
    @DeleteMapping("signout")
    public CommonResponse signout(HttpServletRequest request) throws CommonException {
        String authorization = request.getHeader("Authorization");
        String token = StringUtils.replace(authorization, "bearer ", "");

        if (!consumerTokenServices.revokeToken(token)) {
            throw new CommonException("退出登录失败");
        }
        return new CommonResponse().message("退出登录成功");
    }
}

```



## 2、验证四种授权模式

我么前面已经搭建好了认证服务器，并且集成了`Oauth2`。我们来验证一下：

### 2.1 授权码模式

第一步获取code：

浏览器访问：`http://localhost:8101/oauth/authorize?response_type=code&client_id=kiki&redirect_uri=https://www.baidu.com&state=state`

这个请求会被`SecurityConfigure`类拦截，并重定向到 登录页，要求用户登录：

![image-20200930210943340](/images/activiti6-27/image-20200930210943340.png)

输入任意用户名、密码为123456，登录后系统会携带`code`跳转到我们一开始定义好的redirectUris地址上

![image-20201001085721083](/images/activiti6-27/image-20201001085721083.png)

通过code向认证服务器申请token:

![image-20201001085808205](/images/activiti6-27/image-20201001085808205.png)可以看到我们成功的获取到了token。我们试着通过token获取当前用户信息：

![image-20201001085950332](/images/activiti6-27/image-20201001085950332.png)

如果我们此处不使用token,则会提示错误信息：

![image-20201001090045731](/images/activiti6-27/image-20201001090045731.png)

可以看到提示我们需要授权才可以访问资源。

我们这里验证一下 退出登录。

我们先调用退出登录接口，然后再次获取资源信息：

![image-20201001091128400](/images/activiti6-27/image-20201001091128400.png)

![image-20201001091210217](/images/activiti6-27/image-20201001091210217.png)

可以看到，注销后再次获取资源信息，会提示token无效。

### 2.2 密码模式

密码模式比起授权码模式来说要简单一些。省略了获取 code的环节，提供用户名密码即可。

![image-20201001091618001](/images/activiti6-27/image-20201001091618001.png)

我们使用token获取一下资源：

![image-20201001091732658](/images/activiti6-27/image-20201001091732658.png)

可以看到是没有问题的，这里需要注意的一点就是，实际产生环节中，密码模式下获取token的操作往往是由资源客户端来完成的，这样一来，资源客户端就会获取到你在认证服务器上注册用户的密码。所以密码模式适用于资源客户端可信的应用。

### 2.3 客户端模式

客户端模式（client_credentials）直接根据client的id和密钥即可获取token，无需用户参与。这种模式比较合适消费api的后端服务，此模式不支持refresh token。

![image-20201001092223108](/images/activiti6-27/image-20201001092223108.png)

可以看到返回值中不包含`refresh_token`

#### 2.4 简化模式

以上的3种模式都是需要后端协助的。如果有一个应用，是个纯前端应用，那么如果获取token呢？这就用到了简化模式。

简化模式模式（implicit）比授权码模式少了code环节，回调url直接携带token，这种模式基于安全性考虑，建议把token时效设置短一些，不支持refresh token

直接浏览器输入`http://localhost:8101/oauth/authorize?response_type=token&client_id=kiki&redirect_uri=https://www.baidu.com&state=state`

![image-20201001092702211](/images/activiti6-27/image-20201001092702211.png)

![image-20201001092727344](/images/activiti6-27/image-20201001092727344.png)

可以看到，回调地址栏中携带了token 信息。

