---
title: spring security oauth2 基础知识
date: 2019-12-08 17:12:08
tags: [OAuth2,Spring-Security]
categories: [OAuth2,Spring-Security]
description: spring security oauth2 基础知识
typora-root-url: ..
---

{% note info %} OAuth（开放授权）是一个开放标准，允许用户授权第三方移动应用访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方移动应用或分享他们数据的所有内容，OAuth2.0是OAuth协议的延续版本，但不向后兼容OAuth 1.0即完全废止了OAuth1.0。{% endnote %}

{% note info %} OAuth是一种用来规范令牌（Token）发放的授权机制，主要包含了四种授权模式：授权码模式、简化模式、密码模式和客户端模式。Spring Security OAuth2对这四种授权模式进行了实现.{% endnote %}

## 名词定义

（1）Third-party application：第三方应用程序，本文中又称"客户端"（client）。
（2）HTTP service：HTTP服务提供商，本文中简称"服务提供商"。
（3）Resource Owner：资源所有者。
（4）User Agent：用户代理，本文中就是指浏览器。
（5）Authorization server：认证服务器，即服务提供商专门用来处理认证的服务器。
（6）Resource server：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。

目前只用到过授权码模式、密码模式。另外两种不怎么常见。

## 授权码模式
授权码模式（authorization code）是功能最完整、流程最严密的授权模式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。

![授权码模式](/images/oauth2/授权码模式.png) 

*步骤如下：
（A）用户打开客户端以后，客户端要求用户给予授权。
（B）用户同意给予客户端授权。
（C）客户端使用上一步获得的授权，向认证服务器申请令牌。
（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
（E）客户端使用令牌，向资源服务器申请获取资源。
（F）资源服务器确认令牌无误，同意向客户端开放资源。

1. A步骤中，客户端申请认证的URI，包含以下参数：
*    response_type：表示授权类型，必选项，此处的值固定为"code"
*    client_id：表示客户端的ID，必选项
*    redirect_uri：表示重定向URI，可选项
*    scope：表示申请的权限范围，可选项
*    state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

2. C步骤中，服务器回应客户端的URI，包含以下参数：
*    code：表示授权码，必选项。该码的有效期应该很短，通常设为10分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一 一对应关系。
*    state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

3. D步骤中，客户端向认证服务器申请令牌的HTTP请求，包含以下参数：
*    grant_type：表示使用的授权模式，必选项，此处的值固定为"authorization_code"。
*    code：表示上一步获得的授权码，必选项。
*    redirect_uri：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致。
*    client_id：表示客户端ID，必选项。
*    client_id：表示客户端ID，必选项。

## 密码模式
   密码模式（Resource Owner Password Credentials Grant）中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向认证服务器索要授权。

![密码模式](/images/oauth2/密码模式.png)

步骤如下：
（A）用户向客户端提供用户名和密码。
（B）客户端将用户名和密码发给认证服务器，向后者请求令牌。
（C）认证服务器确认无误后，向客户端提供访问令牌。

B步骤中，客户端发出的HTTP请求，包含以下参数：
* grant_type：表示授权类型，此处的值固定为”password”，必选项。
* username：表示用户名，必选项。
* password：表示用户的密码，必选项。
* scope：表示权限范围，可选项。

## 客户端模式
 这种模式直接根据client的id和密钥即可获取token，无需用户参与。这种模式比较合适消费api的后端服务，不支持refresh token

## 简化模式（implicit）
 这种模式比授权码模式少了code环节，回调url直接携带token，这种模式的使用场景是基于浏览器的应用
 这种模式基于安全性考虑，建议把token时效设置短一些，不支持refresh token

## Spring Security OAuth2
Spring框架对OAuth2协议进行了实现，
Spring Security OAuth2主要包含认证服务器和资源服务器这两大块的实现：

![spring-security-oauth2](/images/oauth2/spring-security-oauth2.png)

认证服务器主要包含了四种授权模式的实现和Token的生成与存储；
资源服务器主要是在Spring Security的过滤器链上加了OAuth2AuthenticationProcessingFilter过滤器，即使用OAuth2协议发放令牌认证的方式来保护我们的资源。

## 补充知识：关于scope参数的说明（小黄人语录）
{% note success %}
 a服务 resourceId ： a
 b服务 resourceId ： b

 Client1：clientid: client1 ,scopse: read,      resourceId:a,b
 Client2：clientid: client2 ,scopse: read,write,resourceId:a,b

 a服务 某个方法加 @PreAuthorize("#oauth2.hasScope('read')")
 b服务 某个方法加 @PreAuthorize("#oauth2.hasScope('write')")

 那么: client1这个客户端就只能访问a服务和b服务的加了read标识的方法
       client2这个client就能同时访问加了read 和 write的方法

 每个服务都应该有个 resourceId，新建 clientDetails 的时候也可以指定 resourceId，不指定就能访问所有，指定了只能访问指定的，访问其他的就会抛异常
 {% endnote %}


*下一章演示代码实例*

参考链接：
[https://mrbird.cc/Spring-Security-OAuth2-Guide.html](https://mrbird.cc/Spring-Security-OAuth2-Guide.html)
[http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)