---
title: Activiti（28） 搭建资源服务器
date: 2020-10-01 09:30:11
tags: [Activiti,Spring-Security-OAuth2]
categories: [Activiti,Spring-Security-OAuth2]
description: Activiti（28） 搭建资源服务器
typora-root-url: ..
---

前面我们已经搭建好了认证服务器，接下来我们将`workflow-activiti-rest`改造为资源服务器。

## 1、引入依赖

首先在`workflow-activiti-rest`的pom 文件中加入以下依赖：

```xml
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

```

## 2、配置资源服务器

资源服务器需要创建**两大配置**：

- 资源服务配置：配置资源信息及访问策略。
- WebSecurity配置：Security配置，**验证token的授权信息**。

### 2.1 创建资源服务配置类

我们在`config`包下新建`ResourceServerConfig`

```java
package com.sxdx.workflow.activiti.rest.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;

/**
 * @program: workflow-activiti
 * @description: 资源服务配置
 * @author: garnett
 * @create: 2020-09-26 17:05
 **/
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    public static final String RESOURCE_ID = "activiti-rest";

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
                .antMatchers("/v2/api-docs", "/swagger-resources/configuration/ui",
                        "/swagger-resources", "/swagger-resources/configuration/security",
                        "/swagger-ui.html", "/webjars/**").permitAll()
                .antMatchers("/modeler/**").permitAll()
                .antMatchers("/**").authenticated();
    }
}

```

类上的`@EnableResourceServer`用于开启资源服务器相关配置。而`ResourceServerSecurityConfigurer`则定义了本服务的`resourceId`。

并且在HttpSecurity中设置了允许无权限访问swagger静态资源和在线流程设计器。

### 2.2 添加token远程服务验证

我们需要在资源服务器中添加配置，用来远程连接认证服务器验证token。在yaml 文件中添加如下内容：

```xml
security:
  oauth2:
    client:
      client-id: kiki #指定了客户端id
      client-secret: 123456 # 指定了客户端密码
    resource:
      token-info-uri: http://127.0.0.1:8101/oauth/check_token #sso客户端token验证，资源服务器就是通过这个向认证服务器校验token是否有效，是否可以访问本resource资源

```

此处指定了client信息和认证服务器token验证接口。`oauth/check_token`是spring-security-ouath自带的，我们直接使用 即可。

### 2.3 创建WebSecurity配置类

`ResourceServerConfig`只能验证token 的准确性，无法验证token中的权限信息，所以我们还需要创建一个WebSecurity配置，我们在config包下新建`SecurityConfig`：

```java
package com.sxdx.workflow.activiti.rest.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

/**
 * @program: workflow-activiti
 * @description: webSecurity配置
 * @author: garnett
 * @create: 2020-09-26 17:28
 **/
@EnableGlobalMethodSecurity(prePostEnabled = true)
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .requestMatchers().antMatchers("/**")
                .and()
                .authorizeRequests()
                .antMatchers("/**").authenticated()
                .and()
                .formLogin().permitAll();
    }
}
```

我们启动项目 ，访问swagger服务：**http://localhost:9090/swagger-ui.html**，任意调用一个接口

![image-20201001102825895](/images/activiti6-28/image-20201001102825895.png)

发现提示无权限访问，说明我们的资源以及被保护起来了。

### 2.4 swagger集成oauth2

我们来改造一下swagger，修改config目录下的`SwaggerConfig`，再次启动项目：

```java
package com.sxdx.workflow.activiti.rest.config;

import io.swagger.annotations.Api;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.*;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spi.service.contexts.SecurityContext;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    private static final String VERSION = "2.0";
    /**
     * 创建API
     */
    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                //指定接口包所在路径
                .apis(RequestHandlerSelectors.basePackage("com.sxdx.workflow.activiti.rest"))
                .paths(PathSelectors.any())
                .build()
                //整合oauth2
                .securitySchemes(Collections.singletonList(apiKey()))
                .securityContexts(Collections.singletonList(securityContext()));
    }

    /**
     * 添加摘要信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .contact(new Contact("Garnett","https://blog.gaoyp.cn/","garnett_boy@sina.com"))
                .title("Springboot2集成Activiti6")
                .description("Springboot2集成Activiti6")
                .termsOfServiceUrl("https://www.kancloud.cn/gaoyipeng/garnett")
                .version(VERSION)
                .build();
    }

    private ApiKey apiKey() {
        return new ApiKey("Bearer", "Authorization", "header");
    }


    /**
     * swagger2 认证的安全上下文
     */
    private SecurityContext securityContext() {
        return SecurityContext.builder()
                .securityReferences(defaultAuth())
                .forPaths(PathSelectors.any())
                .build();
    }

    private List<SecurityReference> defaultAuth() {
        AuthorizationScope authorizationScope = new AuthorizationScope("all", "access_token");
        AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
        authorizationScopes[0] = authorizationScope;
        return Collections.singletonList(new SecurityReference("Bearer",authorizationScopes));
    }
}

```

![image-20201001103306982](/images/activiti6-28/image-20201001103306982.png)

会出现一个`Authorize`授权按钮，我们取到一个token，按照`Bearer token`的格式填进去。

![image-20201001103612812](/images/activiti6-28/image-20201001103612812.png)

再次访问资源

![image-20201001103741105](/images/activiti6-28/image-20201001103741105.png)

可以看到成功获取到了返回信息。

## 3、Activiti自带的Rest接口

截止目前为止，我们已经在`workflow-activiti-rest`项目中成功集成了spring-security-oauth。

其实activiti6为我们提供了很多自带Rest接口，但是需要spring-security的支持，但是我们在一开始为了简单，也为了学习activiti 的Api，就把关于security的部分排除掉了：

```java
@SpringBootApplication(exclude = {
        org.activiti.spring.boot.SecurityAutoConfiguration.class,
        org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration.class
})
```

现在可以把这个改为：

```java
@SpringBootApplication(exclude = {
        org.activiti.spring.boot.SecurityAutoConfiguration.class
})
```

我们依然使用自己的spring-security-oauth。同时修改swagger的配置文件：

```java
    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                //指定接口包所在路径
                .apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
                .paths(PathSelectors.any())
                .build()
                //整合oauth2
                .securitySchemes(Collections.singletonList(apiKey()))
                .securityContexts(Collections.singletonList(securityContext()));
    }
```

重启项目后再次访问swagger.html

![image-20201001105912657](/images/activiti6-28/image-20201001105912657.png)

可以看到这里多了很多接口，这些都是Activiti6自带的，我们可以自由使用。我们做一个测试。

![image-20201001110016069](/images/activiti6-28/image-20201001110016069.png)

可以看到成功获取到了人员信息。