---
title: Activiti（32） 替换Activiti用户和用户组
date: 2020-10-16 16:53:23
tags: [Activiti]
categories: [Activiti]
description: Activiti（32） 替换Activiti用户和用户组
typora-root-url: ..
password: kiki
---

介绍Activiti用户和用户组时我们使用的是activiti自带的用户表，这节我们使用RBAC的用户表代替activiti用户表。

## 1、创建视图

前面我们已经介绍过，此处使用视图来替代原先的用户表

### 1.1 表对应关系

| 系统表                         | activiti表        |
| ------------------------------ | ----------------- |
| base_user(用户表)              | act_id_user       |
| base_role（角色表）            | act_id_group      |
| base_user_role(用户角色对照表) | act_id_membership |

*用户组其实可以看做是我们平常说的角色*。

### 1.2 视图创建SQL

```sql
# 根据 base_user、base_role 和 base_user_role 创建视图
# 需要根据具体的 用户表 和 角色表 进行相应字段的调整

-- 删除自带的用户信息表    

DROP TABLE IF EXISTS ACT_ID_MEMBERSHIP;
DROP TABLE IF EXISTS ACT_ID_GROUP;
DROP TABLE IF EXISTS ACT_ID_USER;
 
-- 如果之前存在视图则删除
    
DROP VIEW  IF EXISTS ACT_ID_MEMBERSHIP;  
DROP VIEW  IF EXISTS ACT_ID_USER;  
DROP VIEW  IF EXISTS ACT_ID_GROUP;
 
CREATE OR REPLACE VIEW act_id_user  AS   
  SELECT  
    u.username    AS ID_,  
    1               AS REV_,  
    u.username     AS FIRST_,  
    ''              AS LAST_,  
    u.email         AS EMAIL_,  
    '000000'      AS PWD_,  
    ''              AS PICTURE_ID_  
  FROM base_user u;    

-- 角色视图
 
CREATE VIEW act_id_group   
AS  
  SELECT r.role_name AS ID_,
         r.rev_ AS REV_,
         r.role_name AS NAME_,
         r.type_ AS TYPE_
  FROM base_role r;  

-- 用户-角色对应视图

CREATE VIEW act_id_membership  
AS  
   SELECT (SELECT u.username FROM base_user u WHERE u.user_id=ur.user_id) AS USER_ID_,
       (SELECT r.role_name FROM base_role r WHERE r.role_id=ur.role_id) AS GROUP_ID_  
   FROM base_user_role ur;
```

可以看到表中只保留act_id_info，另外3张表使用视图代替。

![image-20201019201245069](/images/activiti6-32/image-20201019201245069.png)

### 1.3、禁止自动生成用户表

为了禁止activiti自动生成用户表，我们需要在application.yml中添加如下配置：`db-identity-used: false`

```yaml
# Spring配置
spring:
  # activiti 模块
  # 解决启动报错：class path resource [processes/] cannot be resolved to URL because it does not exist
  activiti:
    check-process-definitions: false
    # 检测身份信息表是否存在
    db-identity-used: false
```



## 2、流程验证

我们修改一下动态表单章节使用的`leave-dynamic-from.bpmn`请假流程。

![image-20201018112505899](/images/activiti6-32/image-20201018112505899.png)

首先我们获取一个token：

![image-20201019214050819](/images/activiti6-32/image-20201019214050819.png)

我们发起一个流程（注意添加请求头）：

![image-20201019201545213](/images/activiti6-32/image-20201019201545213.png)

![image-20201019201526076](/images/activiti6-32/image-20201019201526076.png)

然后`部门经理`获取待办，可以看到已经返回了待办信息：

![image-20201019201812189](/images/activiti6-32/image-20201019201812189.png)

## 3、Spring-Security获取当前操作人

我们在介绍`Activiti`部分时，设置当前流程操作人是通过硬编码的方式：

![image-20201019211817418](/images/activiti6-32/image-20201019211817418.png)

现在我们已经集成了spring-security-oauth2，我们可以通过Spring-Security获取当前操作人。

因为这个是个通用操作，所以我们把这个代码写在workflow-common中。我们首先需要在workflow-common中添加Spring-Security Jar包：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-security</artifactId>
</dependency>
```

然后新建工具类：

```java
package com.sxdx.common.util;

import lombok.extern.slf4j.Slf4j;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.oauth2.provider.authentication.OAuth2AuthenticationDetails;

import java.util.Collection;

/**
 * @program: workflow-activiti
 * @description: 常用工具类
 * @author: garnett
 * @create: 2020-10-18 18:02
 **/
@Slf4j
public class workFlowUtil {

    private static Authentication getOauth2Authentication() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        return authentication;
    }

    /**
     * 获取当前用户名称
     *
     * @return String 用户名
     */
    public static String getCurrentUsername() {
        String name = getOauth2Authentication().getName();
        return name;
    }

    /**
     * 获取当前用户权限集
     *
     * @return Collection<GrantedAuthority>权限集
     */
    public static Collection<GrantedAuthority> getCurrentUserAuthority() {
        return (Collection<GrantedAuthority>) getOauth2Authentication().getAuthorities();
    }

    /**
     * 获取当前令牌内容
     *
     * @return String 令牌内容
     */
    public static String getCurrentTokenValue() {
        try {
            OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails) getOauth2Authentication().getDetails();
            return details.getTokenValue();
        } catch (Exception ignore) {
            return null;
        }
    }
}

```

然后我们替换原先代码中的获取当前登录人的方法：

```java
UserQueryImpl user = new UserQueryImpl();
user = (UserQueryImpl)identityService.createUserQuery().userId(GlobalConfig.getOperator());
```

修改为：

```java
UserQueryImpl user = new UserQueryImpl();
user = (UserQueryImpl)identityService.createUserQuery().userId(workFlowUtil.getCurrentUsername());
```

此处我们可以选择IDEA的批量替换，快捷键：`ctrl + shift + r`

![image-20201019213906187](/images/activiti6-32/image-20201019213906187.png)

点击`Replace All`即可。之后在修改的文件中引入包依赖：

```jav
import com.sxdx.common.util.workFlowUtil;
```

重启服务，我们再次获取当前操作人的待办信息：

![image-20201019214535147](/images/activiti6-32/image-20201019214535147.png)

返回为空，这是因为我们获取token时，用户名输入的是garnett,我们重新获取token

![image-20201019214748273](/images/activiti6-32/image-20201019214748273.png)

![image-20201019214839079](/images/activiti6-32/image-20201019214839079.png)

再次获取待办信息：

![image-20201019214907604](/images/activiti6-32/image-20201019214907604.png)

成功返回待办信息。