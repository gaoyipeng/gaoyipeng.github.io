---
title: spring security oauth2 授权码模式、密码模式代码实践
date: 2019-12-10 22:15:04
tags: [OAuth2,Spring-Security]
categories: [OAuth2,Spring-Security]
description: spring security oauth2 授权码模式、密码模式代码实践
typora-root-url: ..
---
上一节 [spring security oauth2 基础知识](https://blog.gaoyp.cn/2019/12/08/spring-security-oauth2-01/) 介绍了spring security oauth2 基础知识.

本节实例代码演示,为了之后有一个可供代码实践的环境，本次搭建一个名为 spring-all 的聚合工程，今后不特殊说明，均在此项目下进行演示。

> 环境说明：IDEA2019.2 、jdk 1.8 、maven 3.6.2 

## 搭建 spring-all 

### 创建父级 maven项目： spring-all

![创建spring-all](/images/oauth2/创建spring-all.png)

```
<?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.sxdx</groupId>
       <artifactId>spring-all</artifactId>
       <version>1.0-SNAPSHOT</version>
       <packaging>pom</packaging>
   
       <modules>
           <module>../spring-security-oauth2</module>
       </modules>
   
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.1.6.RELEASE</version>
           <relativePath/> <!-- lookup parent from repository -->
       </parent>
   
       <properties>
           <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
       </properties>
   
       <dependencyManagement>
           <dependencies>
               <dependency>
                   <groupId>org.springframework.cloud</groupId>
                   <artifactId>spring-cloud-dependencies</artifactId>
                   <version>${spring-cloud.version}</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
           </dependencies>
       </dependencyManagement>
   </project>
```

> 指定 packaging为pom，表示这是一个纯聚合模块，无需打包为jar或者war。指定spring-boot版本：2.1.6.RELEASE，spring-cloud版本：Greenwich.SR1

它们的对应关系：

|spring boot|spring cloud|
|:---:|:---:|
|Hoxton | 2.2.x|
|Greenwich | 2.1.x|
|Finchley | 2.0.x|
|Edgware | 1.5.x|
|Dalston | 1.5.x|

### 创建一个 module项目： spring-security-oauth2

![创建security-oauth2.png](/images/oauth2/创建security-oauth2.png)

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
    <artifactId>spring-security-oauth2</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-security-oauth2</name>
    <description>spring-security-oauth2 Demo</description>
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

> lombok说明：因为项目里使用了Lombok注解,除引用lombok jar包外，我们还需要在IDEA里安装Lombok插件。[https://www.cnblogs.com/pcheng/p/10945476.html](https://www.cnblogs.com/pcheng/p/10945476.html)

### 授权码模式（authorization code）

#### 创建一个实体类 

```$xslt
package com.sxdx.spring.security.oauth2.entity;

import lombok.Data;
import org.springframework.security.core.GrantedAuthority;
import java.io.Serializable;
import java.util.Set;

@Data
public class KikiSecurityUser implements Serializable {

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

#### 创建用户名密码校验类

```$xslt
package com.sxdx.spring.security.oauth2.service;

import com.sxdx.spring.security.oauth2.entity.KikiSecurityUser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

/**
 * 校验用户名密码
 */
@Service
public class KikiUserDetailService implements UserDetailsService {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        KikiSecurityUser user = new KikiSecurityUser();
        user.setUsername(username);
        user.setPassword(this.passwordEncoder.encode("123456"));

        return new User(username, user.getPassword(), user.isEnabled(),
                user.isAccountNonExpired(), user.isCredentialsNonExpired(),
                user.isAccountNonLocked(), AuthorityUtils.commaSeparatedStringToAuthorityList("user:add"));
    }
}

```

KikiUserDetailService实现了UserDetailsService接口的loadUserByUsername方法。定义登录逻辑：用户名任意，密码为123456 即可通过spring-security登录。loadUserByUsername方法返回一个UserDetails对象，该对象也是一个接口，
包含一些用于描述用户信息的方法，源码如下：

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

* getAuthorities获取用户包含的权限，返回权限集合，权限是一个继承了GrantedAuthority的对象；
* getPassword和getUsername用于获取密码和用户名；
* isAccountNonExpired方法返回boolean类型，用于判断账户是否未过期，未过期返回true反之返回false；
* isAccountNonLocked方法用于判断账户是否未锁定；
* isCredentialsNonExpired用于判断用户凭证是否没过期，即密码是否未过期；
* isEnabled方法用于判断用户是否可用。

#### 创建一个通用返回类

```$xslt
package com.sxdx.spring.security.oauth2.entity;

import java.io.Serializable;
import java.util.HashMap;

public class KikiResponse extends HashMap<String, Object> implements Serializable {

    private static final long serialVersionUID = 967397361339698151L;
    public KikiResponse message(String message) {
        this.put("message", message);
        return this;
    }

    public KikiResponse data(Object data) {
        this.put("data", data);
        return this;
    }

    @Override
    public KikiResponse put(String key, Object value) {
        super.put(key, value);
        return this;
    }

    public String getMessage() {
        return String.valueOf(get("message"));
    }

    public Object getData() {
        return get("data");
    }
}
```
#### 创建Security配置类

```
package com.sxdx.spring.security.oauth2.config;

import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@EnableWebSecurity
public class KikiSecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * Spring Security内部实现好的 BCryptPasswordEncoder。
     * BCryptPasswordEncoder的特点就是，对于一个相同的密码，每次加密出来的加密串都不同
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

}
```

> KikiSecurityConfig类添加了@EnableWebSecurity注解。并且定义了spring-Security密码验证方式 BCryptPasswordEncoder。

#### 创建认证服务器配置类

```
package com.sxdx.spring.security.oauth2.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;


/**
 * 认证服务器配置
 */
@Configuration
@EnableAuthorizationServer
public class KikiAuthorizationServerConfigurer extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("kiki")
                .secret(passwordEncoder.encode("123456"))
                .authorizedGrantTypes("password", "refresh_token","authorization_code")
                .scopes("all")
                .redirectUris("http://localhost:8001/getCode");
    }
}

```
添加了 @EnableAuthorizationServer来开启认证服务器功能。

重写了 configure(ClientDetailsServiceConfigurer clients) 方法。该方法主要配置了：
* 客户端从认证服务器获取令牌的时候，必须使用client_id为 kiki，client_secret为123456的标识来获取；
* 该client_id支持 password、authorization_code 模式获取令牌，并且可以通过refresh_token来获取新的令牌；
* 在获取client_id为kiki的令牌的时候，scope需指定为all，否则将获取失败；
* 配置了redirectUris，指定了获取code时的重定向地址；

如果需要指定多个client，可以继续使用withClient配置。

#### 创建 KikiOauthController 来获取 code

> 就是定义一个方法供 redirectUris 重定向访问

```
package com.sxdx.spring.security.oauth2.controller;

import com.sxdx.spring.security.oauth2.entity.KikiResponse;
import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@RestController
public class KikiOauthController {
    @GetMapping(value = "/getCode")
    public KikiResponse getCode(HttpServletRequest request, HttpServletResponse response){
        return new KikiResponse()
                .put("code",request.getParameter("code"))
                .put("state",request.getParameter("state"));
    }

    @GetMapping("/authentication")
    public Object authentication(Authentication authentication){
        return authentication;
    }
}

```

#### 演示

1、 浏览器访问如下地址（参数说明参考上一节博客）
http://localhost:8001/oauth/authorize?response_type=code&client_id=kiki&redirect_uri=http://localhost:8001/getCode&state=state
访问后页面如下：

![浏览器登录.png](/images/oauth2/浏览器登录.png)

2、 输入任意用户名，密码为 123456，规则在KikiUserDetailService中已经定义。点击 sign in。

![授权码模式-浏览器人工授权.png](/images/oauth2/授权码模式-浏览器人工授权.png)

3、 选择 Approve ，点击Authorize按钮。

![授权码-返回code.png](/images/oauth2/授权码-返回code.png)

发现我们已经获取到了code，其中state参数，没有具体意义，在浏览器访问时输入，返回code时原样一同返回。

4、 通过code，获取token。使用postman发送如下请求POST请求 localhost:8001/oauth/token

![授权码-获取token.png](/images/oauth2/授权码-获取token.png)

填入获取的code 参数，grant_type为固定值，redirect_uri需要与第1步相同，其余参数在KikiAuthorizationServerConfigurer已经定义。

除此之外还需要设置请求头

![授权码-请求头.png](/images/oauth2/授权码-请求头.png)

key为Authorization，value为Basic加上client_id:client_secret经过base64加密后的值（可以使用http://tool.chinaz.com/Tools/Base64.aspx）

![授权码-加密请求头.png](/images/oauth2/授权码-加密请求头.png)

5、 发送请求，获取token

![授权码-返回token.png](/images/oauth2/授权码-返回token.png)

至此，授权码模式已经结束，可通过获取到的token访问资源服务器。一个授权码只能获取一次token，再次访问将会报错。

```
{
    "error": "invalid_grant",
    "error_description": "Invalid authorization code: a8VfVM"
}
```

6、通过token获取资源

![token获取资源401.png](/images/oauth2/token获取资源401.png)

虽然令牌是正确的，但是无法访问/authentication，所以我们必须配置资源服务器，让客户端可以通过合法的令牌来获取资源。
资源服务器的配置也很简单，只需要在配置类上使用@EnableResourceServer注解标注即可：

新建资源服务器配置类

```
package com.sxdx.spring.security.oauth2.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;


@Configuration
@EnableResourceServer
public class KikiResourceServerConfig extends ResourceServerConfigurerAdapter {
   
}

```
但在添加资源服务器后重启服务，再次重复请求过程，返回：

![配置Order.png](/images/oauth2/配置Order.png)

这是因为security配置服务器与资源服务器有加载优先级，需要确保security配置服务器先于资源服务器加载。我们按Ctrl点击 @EnableResourceServer，再点击 @Import(ResourceServerConfiguration.class)
可以看到资源服务器默认Order是3

![资源服务器Order.png](/images/oauth2/资源服务器Order.png)

所以我们在 KikiSecurityConfig 上添加@Order(2)即可。spring 中数字越小说明优先级越高。完整的 KikiSecurityConfig 如下：

```
package com.sxdx.spring.security.oauth2.config;

import org.springframework.context.annotation.Bean;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;


@Order(2)
@EnableWebSecurity
public class KikiSecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * Spring Security内部实现好的 BCryptPasswordEncoder。
     * BCryptPasswordEncoder的特点就是，对于一个相同的密码，每次加密出来的加密串都不同
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

```

再次重复以上步骤：

![authentication请求.png](/images/oauth2/authentication请求.png)


### 密码模式（resource owner password credentials）

 密码模式比起授权码模式来说，相对简单些。我们在postman中请求：localhost:8001/oauth/token?grant_type=password&username=garnett&password=123456

 ![unsupported_grant_type.png](/images/oauth2/unsupported_grant_type.png)

 这是因为密码模式需要用到 AuthenticationManager。

#### 注入AuthenticationManager

```
package com.sxdx.spring.security.oauth2.config;
import org.springframework.context.annotation.Bean;
import org.springframework.core.annotation.Order;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;


@Order(2)
@EnableWebSecurity
public class KikiSecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * Spring Security内部实现好的 BCryptPasswordEncoder。
     * BCryptPasswordEncoder的特点就是，对于一个相同的密码，每次加密出来的加密串都不同
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * 密码模式需要用到
     * @return
     * @throws Exception
     */
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

}


```
> 注入了authenticationManagerBean 对象。

#### 修改认证服务器

接下来修改KikiAuthorizationServerConfigurer认证服务器：

```
package com.sxdx.spring.security.oauth2.config;

import com.sxdx.spring.security.oauth2.service.KikiUserDetailService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;

/**
 * 认证服务器配置
 */

@EnableAuthorizationServer
@Configuration
public class KikiAuthorizationServerConfigurer extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private KikiUserDetailService userDetailService;

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints)
            throws Exception {
        endpoints.authenticationManager(authenticationManager).
        userDetailsService(userDetailService);
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("kiki")
                .secret(passwordEncoder.encode("123456"))
                .authorizedGrantTypes("password", "refresh_token","authorization_code")
                .scopes("all")
                .redirectUris("http://localhost:8001/getCode");
    }
}


```
> 重写了configure(AuthorizationServerEndpointsConfigurer endpoints) 。

#### 演示

重启服务，再次获取token：

 ![密码模式-获取token.png](/images/oauth2/密码模式-获取token.png)

 通过token获取资源：

  ![密码模式-token获取资源.png](/images/oauth2/密码模式-token获取资源.png)

在这里发现一个问题，就是上面密码模式获取到了对应的资源，我一开始以为万事大吉了。
当再次启动服务并先通过密码模式获取token，继而通过token获取资源/authentication时发现再次报401错误了。多次试验发现规律：
先用密码模式的token获取资源就会报401。
先用授权码模式获取的token来请求资源没问题，这个之后再通过密码模式获取资源就也变正常了。

解决方法：在KikiSecurityConfig中添加如下配置：

```
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.requestMatchers()
                .antMatchers("/oauth/**")
                .and()
                .authorizeRequests()
                .antMatchers("/oauth/**").authenticated()
                .and()
                .csrf().disable();
    }
```
至此，密码模式介绍完毕！

不过添加这个后申请授权码页面会报403，所以需要确认下授权码模式和密码模式是否可以共存。待研究后再更新文章。

关于上面的问题，各种尝试后终于解决：修改ikiSecurityConfig中添加如下配置：

```
   http.requestMatchers()
                   .antMatchers("/oauth/**","/login/**","/logout/**")
                   .and()
                   .authorizeRequests()
                   .antMatchers("/oauth/**").authenticated()
                   .and()
                   .csrf().disable()
                   .formLogin().permitAll();
```

至此，我们的项目已经同时支持授权码模式、密码模式了。

### 客户端模式（client credentials）

这里简单介绍下客户端模式：这种模式直接根据client的id和密钥即可获取token，无需用户参与。这种模式比较合适消费api的后端服务，不支持refresh token。

我们修改KikiAuthorizationServerConfigurer 的configure(ClientDetailsServiceConfigurer clients)

```
clients.inMemory()
        .withClient("kiki")
        .secret(passwordEncoder.encode("123456"))
        .authorizedGrantTypes("password", "refresh_token","authorization_code","client_credentials")
        .autoApprove(true)
        .scopes("all")
        .redirectUris("http://localhost:8001/getCode")
```
添加kiki这个client对client_credentials的支持，但是注意：client_credentials和refresh_token是互斥的。以下演示可以证明：

#### 演示

通过postman获取token：http://localhost:8001/oauth/token?grant_type=client_credentials&client_id=kiki&state=state

![客户端模式获取token.png](/images/oauth2/客户端模式获取token.png)

（别忘记添加Authorization请求头）其中 grant_type 固定为 client_credentials，返回的token信息与授权码、密码模式获取的比较发现少了refresh_token。
这也证明了client_credentials模式不支持refresh_token。

接下来通过token获取资源：

![客户端模式token获取资源.png](/images/oauth2/客户端模式token获取资源.png)

### 授权码模式扩展

#### 使用自定义登录页

#### 


### 总结

现在我们的项目就同时支持授权码模式、密码模式以及客户端模式了。一般来说，安全性最好的是授权码模式
即外部系统访问我们的系统使用授权码模式（无需提供给他们用户名密码），内部可信系统直接使用密码模式、或者客户端模式即可。

## postman oauth2 调试使用技巧

在授权码调试过程中，我们需要用到浏览器和postman。来回切换十分麻烦。其实postman提供了很好的调试方法：

![postman-oauth2-授权码1.png](/images/oauth2/postman-oauth2-授权码1.png)

点击 Get New Access Token 按钮配置获取授权码获取token。

![postman-oauth2-授权码2.png](/images/oauth2/postman-oauth2-授权码2.png)

按引导操作即可模拟操作。这个稍微研究下就会了，密码模式大同小异，不再演示了。

参考链接：
[https://mrbird.cc/Spring-Security-OAuth2-Guide.html](https://mrbird.cc/Spring-Security-OAuth2-Guide.html)
[http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)


