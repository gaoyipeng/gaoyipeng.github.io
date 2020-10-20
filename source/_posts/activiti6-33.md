---
title: Activiti（33） Spring-Security获取当前操作人信息
date: 2020-10-19 09:04:04
tags: [Activiti]
categories: [Activiti]
description: Activiti（33） Spring-Security获取当前操作人信息
typora-root-url: ..
password: kiki
---

## 1、SpringSecurity获取当前操作人

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

## 2、OauthUserDetailService改造

我们在集成spring-security-oauth2时，写过一个OauthUserDetailService用来验证用户名密码的正确性。如下：

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

此处的处理逻辑是：用户名任意，密码是123456即可，**我们现在修改为从数据库获取用户信息**。

### 2.1 集成代码生成器

我们这里集成代码生成器，生成基础代码：

![image-20201020183207469](/images/activiti6-33/image-20201020183207469.png)

其他修改之前已经介绍过，这里不再赘述。

### 2.2 代码修改

我们修改`OauthUserDetailService`内容如下：

```java
package com.sxdx.workflow.auth.service;

import com.sxdx.common.entity.AuthUser;
import com.sxdx.workflow.auth.entity.Menu;
import com.sxdx.workflow.auth.entity.SecurityUser;
import com.sxdx.workflow.auth.entity.User;
import com.sxdx.workflow.auth.mapper.MenuMapper;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class OauthUserDetailService implements UserDetailsService {

    @Autowired
    private IUserService iUserService;

    @Autowired
    private IMenuService iMenuService;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		
        User user = iUserService.findByName(username);
        if (user != null) {
            //获取当前用户的权限信息
            List<Menu> permissions = iMenuService.findUserPermissionsByUserId(user.getUserId());
            String userPermissions = permissions.stream().map(Menu::getPerms).collect(Collectors.joining(","));

            boolean notLocked = false;
            if (StringUtils.equals(user.STATUS_VALID, user.getStatus())) {
                notLocked = true;
            }
            String password = user.getPassword();
            AuthUser authUser = new AuthUser(user.getUsername(), password, true, true, true, notLocked,
                    AuthorityUtils.commaSeparatedStringToAuthorityList(userPermissions));
            return authUser;
        }else {
            throw new UsernameNotFoundException("用户名不存在");
        }

        /*return new AuthUser(username, user.getPassword(), user.isEnabled(),
                user.isAccountNonExpired(), user.isCredentialsNonExpired(),
                user.isAccountNonLocked(), AuthorityUtils.commaSeparatedStringToAuthorityList("user:add"));*/
    }
}

```

以上代码实现了获取token时对用户的校验。其中`AuthUser`代码如下：

```java
package com.sxdx.common.entity;
import lombok.Data;
import lombok.EqualsAndHashCode;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.User;

import java.util.Collection;
import java.util.Date;
/**
 * @program: workflow-activiti
 * @description:
 * @author: garnett
 * @create: 2020-10-18 18:38
 **/
@Data
@SuppressWarnings("all")
@EqualsAndHashCode(callSuper = true)
public class AuthUser extends User {

    private static final long serialVersionUID = -6411066541689297219L;
    private Long userId;
    private String avatar;
    private String email;
    private String mobile;
    private String sex;
    private Long deptId;
    private String deptName;
    private String roleId;
    private String roleName;
    private Date lastLoginTime;
    private String description;
    private String status;
    private String deptIds;
    public AuthUser(String username, String password, Collection<? extends GrantedAuthority> authorities) {
        super(username, password, authorities);
    }
    public AuthUser(String username, String password, boolean enabled, boolean accountNonExpired, boolean credentialsNonExpired, boolean accountNonLocked, Collection<? extends GrantedAuthority> authorities) {
        super(username, password, enabled, accountNonExpired, credentialsNonExpired, accountNonLocked, authorities);
    }
}
```



## 3、验证

我们通过密码模式获取token：

![image-20201020185316069](/images/activiti6-33/image-20201020185316069.png)

在代码中打断点：

![image-20201020185226279](/images/activiti6-33/image-20201020185226279.png)

可以看到，通过数据库验证了登录信息。我们验证一下token：

![image-20201020185533531](/images/activiti6-33/image-20201020185533531.png)

返回的信息符合预期。