---
title: Activiti（29） 添加JWT支持
date: 2020-10-01 13:19:58
tags: [Activiti,Spring-Security-OAuth2]
categories: [Activiti,Spring-Security-OAuth2]
description: Activiti（29） 添加JWT支持
typora-root-url: ..
---

## 1、为什么使用JWT

前面我们已经实现了基于token的方式来请求资源，并且添加了资源服务器的token验证配置：

```xml
security:
  oauth2:
    client:
      client-id: kiki #指定了客户端id
      client-secret: 123456 # 指定了客户端密码
    resource:
      token-info-uri: http://127.0.0.1:8101/oauth/check_token
```

也就是说每次请求，资源服务器都会将携带的token通过token-info-uri来向认证服务器验证 token。在请求数较少的时候这不是问题，但是用户量很大时，就会对系统造成一定的负担。我们可以通过JWT令牌的方式解决这个问题。至于什么是JWT，网上一搜一大把，我就不做介绍了。

### 1.1 JWT令牌的优缺点

优点：

- jwt是基于json格式的，容易解析。
- 可以在令牌中自定义内容，具有很好的扩展性。
- 支持对称、非对称加密算法及数字签名技术，JWT防止篡改，安全系数高。
- 在资源服务器中使用jwt可以不依赖认证服务器即可完成验证。
- 服务端无需存储 token，交由客户端保存，节省空间

缺点：

- `jwt`令牌较长，并且会随着内容的添加而变长，占用空间较大。

### 1.2 添加依赖

我们分别在`workflow-auth`、`workflow-activiti-rest`服务中添加如下依赖：

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-jwt</artifactId>
    <version>1.0.10.RELEASE</version>
</dependency>
```

## 2、 添加JWT支持

### 2.1 修改认证服务器

我们先修改`TokenConfig`,之前是`InMemoryTokenStore`内存中存储token，现在改为JWT方式，此处采用对称加密方式来生成jwt令牌，秘钥是123456.

```java
package com.sxdx.workflow.auth.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.InMemoryTokenStore;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.JwtTokenStore;

@Configuration
public class TokenConfig {
    /*@Bean
    public TokenStore tokenStore() {
        return new InMemoryTokenStore();
    }*/

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }


    @Primary
    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("123456");//秘钥
        return converter;
    }

    @Bean
    public TokenEnhancerConfig tokenEnhancer() {
        return new TokenEnhancerConfig();
    }

}

```

然后新建一个`TokenEnhancerConfig`类，用来设置令牌自定义声明:

```java
package com.sxdx.workflow.auth.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.common.DefaultOAuth2AccessToken;
import org.springframework.security.oauth2.common.OAuth2AccessToken;
import org.springframework.security.oauth2.provider.OAuth2Authentication;
import org.springframework.security.oauth2.provider.token.TokenEnhancer;

import java.util.HashMap;
import java.util.Map;

/**
 * @program: workflow-activiti
 * @description: 令牌自定义声明
 * @author: garnett
 * @create: 2020-10-04 11:11
 **/
public class TokenEnhancerConfig implements TokenEnhancer {

    /**
     * 令牌增强：添加jwt令牌中的自定义信息
     * @param accessToken
     * @param authentication
     * @return
     */
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
        Map<String, Object> additionalInfo = new HashMap<>();
        additionalInfo.put("customInfo", authentication.getName() + accessToken.getTokenType());
        ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInfo);
        return accessToken;
    }
}
```

修改认证服务器配置类`AuthorizationServerConfig`中的令牌管理服务方法：设置`TokenEnhancer`

```java
@Autowired
private JwtAccessTokenConverter jwtAccessTokenConverter;
@Autowired
private TokenEnhancerConfig tokenEnhancer;

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

    TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
    tokenEnhancerChain.setTokenEnhancers(Arrays.asList(tokenEnhancer, jwtAccessTokenConverter));
    tokenServices.setTokenEnhancer(tokenEnhancerChain);

    return tokenServices;
}
```

重启服务后再次尝试获取token：

![image-20201005111218985](/images/activiti6-29/image-20201005111218985.png)

可以看到，返回了jwt格式的token，我们验证一下token中的内容：

![image-20201005111338707](/images/activiti6-29/image-20201005111338707.png)

可以看到自定义信息也已经包含了。我们的`workflow-auth`即是认证服务器又是资源服务器，我们使用新获取的token来获取一下`workflow-auth`的资源接口：

![image-20201005111649079](/images/activiti6-29/image-20201005111649079.png)

可以正常获取。

### 2.2 修改资源服务器

接下来我们修改资源服务器`workflow-activiti-rest`的配置 。首先将认证服务器的`TokenConfig.java`复制到资源服务器中（内容完全一样）：

![image-20201005112023470](/images/activiti6-29/image-20201005112023470.png)

然后修改`ResourceServerConfig`资源配置类：将`tokenService`注入到`ResourceServerSecurityConfigurer`中

```java
 @Autowired
private DefaultTokenServices tokenService;

@Override
public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
    resources.resourceId(RESOURCE_ID)//重点，设置资源id
        .stateless(true)
        .tokenServices(tokenService);
}

```

同时我们将application.yaml中的以下配置（远程token校验)去掉：

```xml
security:
  oauth2:
    client:
      client-id: kiki #指定了客户端id
      client-secret: 123456 # 指定了客户端密码
    resource:
      token-info-uri: http://127.0.0.1:8101/oauth/check_token #sso客户端token验证，资源服务器就是通过这个向认证服务器校验token是否有效，是否可以访问本resource资源
```

重启服务，访问swagge服务，输入我们生成的token:

![image-20201005120644835](/images/activiti6-29/image-20201005120644835.png)

![image-20201005120725589](/images/activiti6-29/image-20201005120725589.png)

可以看到我们顺利获取到了资源，说明token是通过了验证，无需调用认证服务器即可验证token的正确性。