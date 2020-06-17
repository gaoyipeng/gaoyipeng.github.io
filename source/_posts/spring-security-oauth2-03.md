---
title: spring security oauth2 SSO 单点登录及自定义令牌配置
date: 2019-12-14 21:58:18
tags: [OAuth2,Spring-Security]
categories: [OAuth2,Spring-Security]
description: spring security oauth2 SSO 单点登录及自定义令牌配置
typora-root-url: ..
---

上一节 [spring security oauth2 授权码模式、密码模式代码实践](https://blog.gaoyp.cn/2019/12/10/spring-security-oauth2-02/) 介绍了spring security oauth2 的3种模式。
本节我们将在上一节的基础上实现SSO单点登录，及自定义自定义令牌配置。SSO单点登录的概念就不做解释了，请自行百度。

## 自定义令牌配置

上一节我们在认证服务器 spring-security-oauth2 的资源配置类 KikiAuthorizationServerConfigurer 中定义了 ClientDetails：

```
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("kiki")
                .secret(passwordEncoder.encode("123456"))
                .authorizedGrantTypes("password", "refresh_token","authorization_code","client_credentials")
                .autoApprove(true)
                .scopes("all")
                .redirectUris("http://localhost:8001/getCode");
    }
```
指定了 client-id为kiki ，client-secret为123456，scopes范围为all,以及redirectUris。并且配置了它支持的认证类型：

* "authorization_code"：授权码模式
* "password"：密码模式
* client_credentials:客户端模式
* "refresh_token"：刷新token


现在我们修改认证服务器 KikiAuthorizationServerConfigurer如下：

```
clients.inMemory()
            .withClient("kiki")
            .secret(passwordEncoder.encode("123456"))
            .authorizedGrantTypes("password", "refresh_token","authorization_code","client_credentials")
            .autoApprove(true)
            .scopes("all")
            .redirectUris("http://localhost:8001/getCode")
            .resourceIds("kiki-resource")
        .and()
            .withClient("client1")
            .secret(passwordEncoder.encode("123456"))
            .authorizedGrantTypes("authorization_code", "password","refresh_token")
            .autoApprove(true)
            .scopes("read")
            .redirectUris("http://localhost:8002/one/login")
            .resourceIds("client1-resource")
        .and()
            .withClient("client2")
            .secret(passwordEncoder.encode("123456"))
            .authorizedGrantTypes("authorization_code","password", "refresh_token")
            .autoApprove(true)
            .scopes("read","write")
            .redirectUris("http://localhost:8003/two/login")
            .resourceIds("client2-resource");
```

可以看到，我们新增了2个 Client，分别为client1，client2.并指定它们的认证类型为："authorization_code","password", "refresh_token"。
* autoApprove(true)：表示在授权码模式中，用户无需进入授权页面手动点击授权按钮（Authorize）,直接返回code。
* scopes：指定了Client所能访问的范围。client1可以访问带有@PreAuthorize("#oauth2.hasScope('read')")的资源，client2可以访问带有@PreAuthorize("#oauth2.hasScope('read')")和
  @PreAuthorize("#oauth2.hasScope('write')")的资源。如果没有@PreAuthorize，则都可以访问。
* redirectUris：配置了重定向URL，否则授权码模式输入用户名密码后会报错：error="invalid_request", error_description="At least one redirect_uri must be registered with the client."
* resourceIds：指定了Client 所能访问的资源ID。client1可以访问带有resourceId为client1-resource的资源，client2可以访问带有resourceId为client2-resource的资源，

对于scopes和resourceIds不熟悉的，可以参考 [https://blog.gaoyp.cn/2019/12/08/spring-security-oauth2-01/](https://blog.gaoyp.cn/2019/12/08/spring-security-oauth2-01/)


## 新建2个资源服务器

为了实现功能，我们需要新建2个资源服务器。现约定如下（于认证服务器的clients配置相对应）：

|系统名称|端口|client-id|client-secret|
|:---:|:---:|:---:|:---:|
|sso-resource-one  |8002 |client1 |123456 |
|sso-resource-two  |8003 |client2 |123456 |

### 新建资源服务器sso-resource-one：

新建一个module，pom.xml如下:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.sxdx</groupId>
        <artifactId>spring-all</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../spring-all/pom.xml</relativePath>
    </parent>
    <groupId>com.sxdx</groupId>
    <artifactId>sso-resource-one</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>sso-resource-one</name>
    <description>资源服务器和sso客户端</description>

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
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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

再贴出sso-resource-one的application.yaml：

```
server:
  port: 8002
  servlet:
    context-path: /one
    session:
      cookie:
        name: ONESESSION

security:
  oauth2:
    client:
      client-id: client1 #指定了客户端id
      client-secret: 123456 # 指定了客户端密码
      user-authorization-uri: http://127.0.0.1:8001/oauth/authorize #指定为认证服务器的/oauth/authorize地址
      access-token-uri: http://127.0.0.1:8001/oauth/token # 指定认证服务器的/oauth/token地址
    resource:
      token-info-uri: http://127.0.0.1:8001/oauth/check_token #sso客户端token验证，资源服务器就是通过这个向认证服务器校验token是否有效，是否可以访问本resource资源
      user-info-uri: http://127.0.0.1:8001/authentication #sso客户端获取当前用户信息
```

#### 添加@EnableOAuth2Sso

在启动类上添加@EnableOAuth2Sso，表面这是一个SSO客户端，开启SSO的支持。

```
package com.sxdx.sso.resource.one;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.security.oauth2.client.EnableOAuth2Sso;

@EnableOAuth2Sso
@SpringBootApplication
public class SsoResourceOneApplication {
    public static void main(String[] args) {
        SpringApplication.run(SsoResourceOneApplication.class, args);
    }
}
```

#### 新建WebSecurity配置类

新建config包，在新建config包下新建WebSecurityConfigurer类

```
package com.sxdx.sso.resource.one.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Order(101)
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.requestMatchers()
                .antMatchers("/**")
                .and()
                .authorizeRequests()
                .antMatchers("/**").authenticated()
                .and()
                .csrf().disable()
                .formLogin().permitAll();
    }
}

```

*   设置了资源服务的WebSecurity配置，所有请求均需要认证后才可访问。此处配置了 @Order(101)，否则会报错

```
Caused by: java.lang.IllegalStateException: @Order on WebSecurityConfigurers must be unique. Order of 100 was already used on com.sxdx.sso.resource.one.config.WebSecurityConfigurer$$EnhancerBySpringCGLIB$$2b177f7a@48bc2fce, so it cannot be used on org.springframework.boot.autoconfigure.security.oauth2.client.OAuth2SsoDefaultConfiguration$$EnhancerBySpringCGLIB$$ff13e17c@1eca3ea7 too.
	at org.springframework.security.config.annotation.web.configuration.WebSecurityConfiguration.setFilterChainProxySecurityConfigurer(WebSecurityConfiguration.java:148) ~[spring-security-config-5.1.5.RELEASE.jar:5.1.5.RELEASE]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_171]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_171]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_171]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_171]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredMethodElement.inject(AutowiredAnnotationBeanPostProcessor.java:708) ~[spring-beans-5.1.8.RELEASE.jar:5.1.8.RELEASE]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90) ~[spring-beans-5.1.8.RELEASE.jar:5.1.8.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:374) ~[spring-beans-5.1.8.RELEASE.jar:5.1.8.RELEASE]
	... 17 common frames omitted
```
这是因为认证服务器的WebSecurityConfigurers已经默认使用了@Order(100)，其实我们在认证服务器中已经改为 @Order(2)，但这里还是不能使用 @Order(100)。具体原因不明。反正改为@Order(101)就好了。

*   @EnableGlobalMethodSecurity(prePostEnabled = true)，开启权限注解，否则无法使用@PreAuthorize。

#### 新建ResourceServerConfig资源配置类

在config包下，新建ResourceServerConfig：

````
package com.sxdx.sso.resource.one.config;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;


@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter{

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.resourceId("client1-resource").stateless(true); //重点，设置资源id
    }

    public void configure(HttpSecurity http) throws Exception {
        http.requestMatchers().antMatchers("/**")
                .and()
                .authorizeRequests()
                .anyRequest().authenticated()
                .antMatchers("/login/**").permitAll();
        ;
    }
}

````
* @EnableResourceServer 开启资源服务器配置
* 定义了本服务的resourceId为client1-resource
* 定义了资源访问规则。



#### 新建测试类

新建OneController：

```
package com.sxdx.sso.resource.one.controller;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.security.Principal;
import java.util.HashMap;
import java.util.Map;

@RestController
public class OneController {

    @PreAuthorize("#oauth2.hasScope('write')")
    @GetMapping("/index")
    public Map<String,String> index(){
        Map<String,String> mp = new HashMap<>();
        mp.put("key","/index");
        return mp;
    }

    @PreAuthorize("hasAuthority('user:add') and #oauth2.hasScope('read')")
    @GetMapping("/user")
    public Principal user(Principal principal) {
        return principal;
    }
}

```
* @PreAuthorize("#oauth2.hasScope('read')"):只有scope包含read才有权访问。
* @PreAuthorize("hasAuthority('user:add') and #oauth2.hasScope('read')"):只有scope包含read，并且用户角色为 user:add 才有权访问。
  认证服务器中KikiUserDetailService定义了所有登录用户，均有user:add权限

```
 @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        KikiSecurityUser user = new KikiSecurityUser();
        user.setUsername(username);
        user.setPassword(this.passwordEncoder.encode("123456"));

        return new User(username, user.getPassword(), user.isEnabled(),
                user.isAccountNonExpired(), user.isCredentialsNonExpired(),
                user.isAccountNonLocked(), AuthorityUtils.commaSeparatedStringToAuthorityList("user:add"));
    }
```

至此资源服务器sso-resource-one搭建完成

### 新建资源服务器module：sso-resource-two

仿照sso-resource-one,新建sso-resource-two即可。不再贴出代码。

## 演示

### resourceIds 作用演示

查询 http://127.0.0.1:8002/one/user。

![获取client1的资源.png](/images/oauth2/获取client1的资源.png)

我们通过postman 密码模式获取token

![获取client1的token.png](/images/oauth2/获取client1的token.png)

点击Send访问，记着添加请求头（client1:123456 base64编码）：

![client1的资源.png](/images/oauth2/client1的资源.png)

可以看到可以获取到对应资源。因为client1对应的resourceID为client1-resource。而 http://127.0.0.1:8002/one/user 就是属于client1-resource的资源。

现在我们再次获取请求资源，不同的是把URl 改为 http://127.0.0.1:8003/two/user 。来验证client1是否可以获取client2-resource的资源。
返回结果：
```
{
    "error": "access_denied",
    "error_description": "Invalid token does not contain resource id (client2-resource)"
}
```
无权访问client2-resource。我们修改client1的resourceIds，并重启服务。
```
.and()
.withClient("client1")
.secret(passwordEncoder.encode("123456"))
.authorizedGrantTypes("authorization_code","password", "refresh_token")
.autoApprove(true)
.scopes("read","write")
.redirectUris("http://localhost:8003/two/login")
.resourceIds("client2-resource","client1-resource")
```
 resourceIds("client2-resource","client1-resource")表示client1可以访问sso-resource-one和sso-resource-two的资源。

 访问http://127.0.0.1:8002/one/user ，发现可以获取资源。修改URL为http://127.0.0.1:8003/two/user.也可以获取到资源。

### scopes 作用演示

再次修改URL为http://127.0.0.1:8002/one/index,
返回：
```
{
    "error": "access_denied",
    "error_description": "不允许访问"
}
```
因为client1 的scope为read,而/one/index 需要 write.

### SSO 单点登录

其实上面的演示过程已经实现了单点登录，我们通过token可以同时获取到sso-resource-one和sso-resource-two的资源,不过这个只适合密码模式。
浏览器直接访问 http://127.0.0.1:8002/one/user 会报Full authentication is required to access this resource。

现在我们把sso-resource-one和sso-resource-two的 ResourceServerConfig类的 @EnableResourceServer注解去掉。
浏览器访问 http://127.0.0.1:8002/one/user,浏览器会重定向到 http://127.0.0.1:8001/login:

![SSO登录.png](/images/oauth2/SSO登录.png) 

输入garnett 123456 点击 “Sign in”

![SSO-client1.png](/images/oauth2/SSO-client1.png) 

可以看到浏览器返回了http://127.0.0.1:8002/one/user对应的资源。接下来我们新开一个页面访问 http://127.0.0.1:8003/two/user

![访问sso-client2.png](/images/oauth2/访问sso-client2.png) 

![sso-client2.png](/images/oauth2/sso-client2.png) 

我们发现，我没要求输入用户名密码，即可获取到对应资源。

我们再访问 http://127.0.0.1:8002/one/index，会报403异常,不是很理解。发现将client1的 scopes改为 ("read","write")后就可以访问了。
哪位知道为啥还请不吝赐教！

![sso-403.png](/images/oauth2/sso-403.png) 

访问 http://127.0.0.1:8003/two/index ，返回结果： {"key":"/index"}

参考：
https://mrbird.cc/Spring-Security-OAuth2-SSO.html
http://www.it1352.com/983633.html

--------------------------------------------------------------------------------------------
学然后知不足,教然后知困。知不足,然后能自反也;知困,然后能自强也。