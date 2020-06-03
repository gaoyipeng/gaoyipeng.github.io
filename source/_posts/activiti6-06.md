---
title: Activiti（6） 用户和用户组
date: 2020-06-03 10:55:25
tags: Activiti
categories: Activiti
description: Activiti（6） 用户和用户组
typora-root-url: ..
password: kiki
---

> 本节我们来介绍一下activiti的用户和用户组。

前面我们已经介绍过activiti的数据库表，涉及到用户和用户组的共4张。

![image-20200603110206442](/images/activiti/activiti6-06/image-20200603110206442.png)

# 1、引出问题

这4张表是`activiti`自动帮我们生成的。如果我们现在已经有一个成熟的系统，拥有自身的用户、权限等表结构。此时想要集成`Activiti`，难道要把用户信息都复制到`Activiti`的用户表中吗?

当然不是，此时就用到了数据库视图技术。

# 2、原用户体系

假设我们现在的用户体系包含如下几张表：

sys_user(用户表)：

```sql
CREATE TABLE `sys_user` (
  `user_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `dept_id` bigint(20) DEFAULT NULL COMMENT '部门ID',
  `login_name` varchar(30) NOT NULL COMMENT '登录账号',
  `user_name` varchar(30) NOT NULL COMMENT '用户昵称',
  `user_type` varchar(2) DEFAULT '00' COMMENT '用户类型（00系统用户）',
  `email` varchar(50) DEFAULT '' COMMENT '用户邮箱',
  `phonenumber` varchar(11) DEFAULT '' COMMENT '手机号码',
  `sex` char(1) DEFAULT '0' COMMENT '用户性别（0男 1女 2未知）',
  `avatar` varchar(100) DEFAULT '' COMMENT '头像路径',
  `password` varchar(50) DEFAULT '' COMMENT '密码',
  `salt` varchar(20) DEFAULT '' COMMENT '盐加密',
  `status` char(1) DEFAULT '0' COMMENT '帐号状态（0正常 1停用）',
  `del_flag` char(1) DEFAULT '0' COMMENT '删除标志（0代表存在 2代表删除）',
  `login_ip` varchar(50) DEFAULT '' COMMENT '最后登陆IP',
  `login_date` datetime DEFAULT NULL COMMENT '最后登陆时间',
  `create_by` varchar(64) DEFAULT '' COMMENT '创建者',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(64) DEFAULT '' COMMENT '更新者',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=110 DEFAULT CHARSET=utf8 COMMENT='用户信息表';
```

sys_role（角色表）：

```sql
CREATE TABLE `sys_role` (
  `role_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '角色ID',
  `role_name` varchar(30) NOT NULL COMMENT '角色名称',
  `role_key` varchar(100) NOT NULL COMMENT '角色权限字符串',
  `role_sort` int(4) NOT NULL COMMENT '显示顺序',
  `data_scope` char(1) DEFAULT '1' COMMENT '数据范围（1：全部数据权限 2：自定数据权限 3：本部门数据权限 4：本部门及以下数据权限）',
  `status` char(1) NOT NULL COMMENT '角色状态（0正常 1停用）',
  `del_flag` char(1) DEFAULT '0' COMMENT '删除标志（0代表存在 2代表删除）',
  `create_by` varchar(64) DEFAULT '' COMMENT '创建者',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(64) DEFAULT '' COMMENT '更新者',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`role_id`)
) ENGINE=InnoDB AUTO_INCREMENT=111 DEFAULT CHARSET=utf8 COMMENT='角色信息表';
```

sys_user_role(用户角色对照表)

```sql
CREATE TABLE `sys_user_role` (
  `user_id` bigint(20) NOT NULL COMMENT '用户ID',
  `role_id` bigint(20) NOT NULL COMMENT '角色ID',
  PRIMARY KEY (`user_id`,`role_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户和角色关联表';
```

# 3、创建视图

## 3.1 表对应关系

| 系统表                        | activiti表        |
| ----------------------------- | ----------------- |
| sys_user(用户表)              | act_id_user       |
| sys_role（角色表）            | act_id_group      |
| sys_user_role(用户角色对照表) | act_id_membership |



## 3.2 创建视图

```sql
# 根据 sys_user、sys_role 和 sys_user_role 创建视图
# 需要根据具体的 用户表 和 角色表 进行相应字段的调整

-- 删除自带的用户信息表    

DROP TABLE IF EXISTS ACT_ID_MEMBERSHIP;
DROP TABLE IF EXISTS ACT_ID_GROUP;
DROP TABLE IF EXISTS ACT_ID_USER;
 
-- 如果之前存在视图则删除
    
DROP VIEW  IF EXISTS ACT_ID_MEMBERSHIP;  
DROP VIEW  IF EXISTS ACT_ID_USER;  
DROP VIEW  IF EXISTS ACT_ID_GROUP;
 
CREATE OR REPLACE VIEW ACT_ID_USER  AS   
  SELECT  
    u.login_name    AS ID_,  
    0               AS REV_,  
    u.user_name     AS FIRST_,  
    ''              AS LAST_,  
    u.email         AS EMAIL_,  
    u.password      AS PWD_,  
    ''              AS PICTURE_ID_  
  FROM sys_user u;    

-- 角色视图
 
CREATE VIEW ACT_ID_GROUP   
AS  
  SELECT r.role_key AS ID_,
         NULL AS REV_,
         r.role_name AS NAME_,
         'assignment' AS TYPE_
  FROM sys_role r;  

-- 用户-角色对应视图

CREATE VIEW ACT_ID_MEMBERSHIP  
AS  
   SELECT (SELECT u.login_name FROM sys_user u WHERE u.user_id=ur.user_id) AS USER_ID_,
       (SELECT r.role_key FROM sys_role r WHERE r.role_id=ur.role_id) AS GROUP_ID_  
   FROM sys_user_role ur;
```

![image-20200603154308303](/images/activiti/activiti6-06/image-20200603154308303.png)

# 4、禁止自动生成用户表

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



OK ,activiti用户和用户组介绍完毕！