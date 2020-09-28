---
title: Activiti（26） Spring-Security-OAuth2简介
date: 2020-09-28 14:01:47
tags: [Activiti,Spring-Security-OAuth2]
categories: [Activiti,Spring-Security-OAuth2]
description: Activiti（26） Spring-Security-OAuth2简介
typora-root-url: ..
---

前面关于Activiti6的介绍，已经告一段落，基本的流程操作步骤都已经有了，后期有什么遗漏再做补充。

之前的流程演示时，我们都是直接使用Swagger或者是Postman来调用接口，没有任何验证过程、无法保证系统安全性。本节我们为系统集成Spring-Security-OAuth2来实现资源的认证和授权。

## 1、Spring-Security-OAuth2简介

Spring-Security-OAuth2分为**Spring-Security**和**OAuth2**两个知识点。Spring-Security-OAuth2提供了这2个知识点的整合。

Spring Security是一个基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架。由于它 是Spring生态系统中的一员，因此它伴随着整个Spring生态系统不断修正、升级，在spring boot项目中加入spring security更是十分简单，使用Spring Security 减少了为企业系统安全控制编写大量重复代码的工作。 

OAuth（开放授权）是一个开放标准协议，允许用户授权第三方应用访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方应用或分享他们数据的所有内容。

OAuth是一种用来规范令牌（Token）发放的授权机制，主要包含了四种授权模式：**授权码模式、简化模式、密码模式和客户端模式**。Spring Security OAuth2对这四种授权模式进行了实现。



## 2、四种授权模式

这里简要的介绍一下4种授权模式及他们适用的范围。

### 2.1 授权码模式

授权码模式（authorization code）是功能最完整、流程最严密的授权模式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。

适用于和外部系统的对接，例如我们经常使用的微信扫码登录、QQ登录、Github登录都是这种模式。

![授权码模式](/images/activiti6-26/%E6%8E%88%E6%9D%83%E7%A0%81%E6%A8%A1%E5%BC%8F.png)

步骤如下：

（A）用户打开客户端以后，客户端要求用户给予授权。
（B）用户同意给予客户端授权。
（C）客户端使用上一步获得的授权，向认证服务器申请令牌。
（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
（E）客户端使用令牌，向资源服务器申请获取资源。
（F）资源服务器确认令牌无误，同意向客户端开放资源。

A步骤中，客户端申请认证的URI，包含以下参数：

*    response_type：表示授权类型，必选项，此处的值固定为"code"
*    client_id：表示客户端的ID，必选项
*    redirect_uri：表示重定向URI，必选项
*    scope：表示申请的权限范围，可选项
*    state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

C步骤中，服务器回应客户端的URI，包含以下参数：

*    code：表示授权码。该码的有效期应该很短，通常设为10分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一 一对应关系。
*    state：如果客户端的请求中包含这个参数，认证服务器的回应也一模一样包含这个参数。

D步骤中，客户端向认证服务器申请令牌的HTTP请求，包含以下参数：

*    grant_type：表示使用的授权模式，必选项，此处的值固定为"authorization_code"。
*    code：表示上一步获得的授权码，必选项。
*    redirect_uri：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致。
*    client_id：表示客户端ID，必选项。
*    client_secret：表示客户端security，必选项。
*    scope：表示申请的权限范围，可选项

### 2.2 密码模式

   密码模式（Resource Owner Password Credentials Grant）中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向认证服务器索要授权。

适用于客户端是自己开发的系统，因为我们会将用户名、密码提供给client客户端，必须要是可信系统才行。

![密码模式](/images/activiti6-26/%E5%AF%86%E7%A0%81%E6%A8%A1%E5%BC%8F.png)

步骤如下：
（A）用户向客户端提供用户名和密码。
（B）客户端将用户名和密码发给认证服务器，向后者请求令牌。
（C）认证服务器确认无误后，向客户端提供访问令牌。

B步骤中，客户端发出的HTTP请求，包含以下参数：

* grant_type：表示授权类型，此处的值固定为”password”，必选项。
* username：表示用户名，必选项。
* password：表示用户的密码，必选项。
* scope：表示权限范围，可选项。

### 2.3 客户端模式

客户端模式（client_credentials）直接根据client的id和密钥即可获取token，无需用户参与。这种模式比较合适消费api的后端服务，此模式不支持refresh token。

![在这里插入图片描述](/images/activiti6-26/20190314171857269.png)

步骤如下：
（A）用户向客户端提供用户名和密码。
（B）客户端将用户名和密码发给认证服务器，向后者请求令牌。

A步骤中，客户端发出的HTTP请求，包含以下参数：

* grant_type：表示授权类型，此处的值固定为”password”，必选项。
* client_id：客户端ID，必选项。
* client_secret：客户端secret，必选项。

### 2.4 简化模式

 简化模式模式（implicit）比授权码模式少了code环节，回调url直接携带token，这种模式的使用场景是基于浏览器的应用
 这种模式基于安全性考虑，建议把token时效设置短一些，不支持refresh token



## 3、scope参数

 a 服务 resourceId为 ： a
 b 服务 resourceId为：  b

 Client1：clientid: client1 ,scopse: read          resourceId:a,b
 Client2：clientid: client2 ,scopse: read,write,resourceId:a,b

 a服务 某个方法加 @PreAuthorize("#oauth2.hasScope('read')")
 b服务 某个方法加 @PreAuthorize("#oauth2.hasScope('write')")

 **那么:** 

client1这个客户端就只能访问a服务和b服务的加了read标识的方法
client2这个client就能同时访问加了read 和 write的方法

 另外每个服务都应该有个 resourceId，新建 clientDetails 的时候可以指定 resourceId，不指定就能访问所有，指定了只能访问指定的，访问其他的就会抛异常。

{% note success %}

看不懂没关系，先看后面的代码实操，还是不懂也没关系，再看几遍就好了。

{% endnote %}



## 4、题外话

关于Spring-Security-OAuth2停止维护的信息：

2018 年 1 月 30 号 Spring 官方发了一个通知，说是要逐渐停止现有项目中的 OAuth2 支持，因为 OAuth2 的落地方案比较混乱，在 Spring Security OAuth、Spring Cloud Security、Spring Boot 1.5.x 以及当时最新的 Spring Security5.x 中都提供了对 OAuth2 的实现。以至于当开发者需要使用 OAuth2 时，不得不问，到底选哪一个依赖合适呢？

所以Spring官方决定在 Spring Security5.3 中构建下一代 OAuth2.0 支持。且建议使用者将应用迁移到Spring Security5.3。

但在 2019.11.14 Spring 官方又发了一个通知，**不再提供对授权服务器的支持**。

许多开发者表示难以接受。这件事也在 Spring 社区引发了激烈的讨论，终于2020.04.15日 Spring 官方宣布启动Spring Authorization Server 项目提供 Authorization Server 支持。并且在2020.08.21  Spring Authorization Server 0.0.1正式发布！

我理解就是**Spring-Security-OAuth2  =  Spring-Security5.3  +  Spring Authorization Server**

虽然现在已经是2020年9月份了，但是我还是决定使用Spring-Security-OAuth2，因为不想采用太新的项目，一定会有bug。后期如果需要升级再说。