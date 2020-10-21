---
title: Activiti（31） RBAC基于角色的访问控制
date: 2020-10-10 11:14:54
tags: [Activiti,RBAC]
categories: [Activiti,RBAC]
description: Activiti（31） RBAC基于角色的访问控制
typora-root-url: ..
password: kiki
---

本节我们在系统中集成RBAC（**R**ole-**B**ased **A**ccess **C**ontrol，基于角色的访问控制）模型。简单来说就是一个用户拥有若干角色，每一个角色拥有若干权限。这样就构造成“用户-角色-权限”的授权模型。在这种模型中，用户与角色之间，角色与权限之间，一般都是多对多的关系。

## 1、表结构设计

![image-20201015100138801](/images/activiti6-31/image-20201015100138801.png)

![image-20201017213317296](/images/activiti6-31/image-20201017213317296.png)

比如获取用户名为 Garnett 的用户权限过程为：

1. 通过`Garnett` 这个user_id从base_user_role表获取对应的role_id的集合；
2. 通过第1步获取的role_id从base_role_menu表获取对应的menu_id的集合；
3. 通过第2步获取的menu_id从base_menu获取menu相关信息（base_menu表的permission为权限信息）。

另外我们补充一张base_dept表来存储组织机构数据。

接下里我们贴出建表SQL：

```sql
DROP TABLE IF EXISTS base_user;
DROP TABLE IF EXISTS base_dept;
DROP TABLE IF EXISTS base_role;
DROP TABLE IF EXISTS base_menu;
DROP TABLE IF EXISTS base_user_role;
DROP TABLE IF EXISTS base_role_menu;

CREATE TABLE `base_user` (
  `user_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `username` varchar(50) NOT NULL COMMENT '用户名',
  `password` varchar(128) NOT NULL COMMENT '密码',
  `dept_id` bigint(20) DEFAULT NULL COMMENT '部门ID',
  `email` varchar(128) DEFAULT NULL COMMENT '邮箱',
  `mobile` varchar(20) DEFAULT NULL COMMENT '联系电话',
  `status` char(1) NOT NULL COMMENT '状态 0锁定 1有效',
  `order_num` double(20,0) DEFAULT NULL COMMENT '排序',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `modify_time` datetime DEFAULT NULL COMMENT '修改时间',
  `last_login_time` datetime DEFAULT NULL COMMENT '最近访问时间',
  `gender` char(1) DEFAULT NULL COMMENT '性别 0男 1女 2保密',
  `avatar` varchar(255) DEFAULT NULL COMMENT '头像',
  `description` varchar(255) DEFAULT NULL COMMENT '描述',
  PRIMARY KEY (`user_id`) USING BTREE,
  UNIQUE KEY `username` (`username`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='用户表';

INSERT INTO `base_user` VALUES (1, 'admin', '$2a$10$v8hx0S0I2XTOMcDnDGhb3uwSWnMgy9ckgVM5x6GfwepuBwPnGv7bK', 1, 'admin@qq.com', '15935887412', '1', 1,'2020-10-10 20:39:22', '2020-10-10 10:18:36', '2020-10-10 15:57:00', '0', 'default.jpg', '我是Garnett');
INSERT INTO `base_user` VALUES (2, 'hruser', '$2a$10$v8hx0S0I2XTOMcDnDGhb3uwSWnMgy9ckgVM5x6GfwepuBwPnGv7bK', 1, 'hruser@qq.com', '15935887412', '1', 1,'2020-10-10 20:39:22', '2020-10-10 10:18:36', '2020-10-10 15:57:00', '0', 'default.jpg', '我是Garnett');
INSERT INTO `base_user` VALUES (3, 'kafeitu', '$2a$10$v8hx0S0I2XTOMcDnDGhb3uwSWnMgy9ckgVM5x6GfwepuBwPnGv7bK', 1, 'kafeitu@qq.com', '15935887412', '1', 1,'2020-10-10 20:39:22', '2020-10-10 10:18:36', '2020-10-10 15:57:00', '0', 'default.jpg', '我是Garnett');
INSERT INTO `base_user` VALUES (4, 'leaderuser', '$2a$10$v8hx0S0I2XTOMcDnDGhb3uwSWnMgy9ckgVM5x6GfwepuBwPnGv7bK', 1, 'leaderuser@qq.com', '15935887412', '1',1, '2020-10-10 20:39:22', '2020-10-10 10:18:36', '2020-10-10 15:57:00', '0', 'default.jpg', '我是Garnett');
INSERT INTO `base_user` VALUES (5, 'manager', '$2a$10$v8hx0S0I2XTOMcDnDGhb3uwSWnMgy9ckgVM5x6GfwepuBwPnGv7bK', 1, 'manager@qq.com', '15935887412', '1', 1,'2020-10-10 20:39:22', '2020-10-10 10:18:36', '2020-10-10 15:57:00', '0', 'default.jpg', '我是Garnett');
INSERT INTO `base_user` VALUES (6, 'thomas', '$2a$10$v8hx0S0I2XTOMcDnDGhb3uwSWnMgy9ckgVM5x6GfwepuBwPnGv7bK', 1, 'thomas@qq.com', '15935887412', '1', 1,'2020-10-10 20:39:22', '2020-10-10 10:18:36', '2020-10-10 15:57:00', '0', 'default.jpg', '我是Garnett');
INSERT INTO `base_user` VALUES (7, 'treasurer', '$2a$10$v8hx0S0I2XTOMcDnDGhb3uwSWnMgy9ckgVM5x6GfwepuBwPnGv7bK', 1, 'treasurer@qq.com', '15935887412', '1', 1,'2020-10-10 20:39:22', '2020-10-10 10:18:36', '2020-10-10 15:57:00', '0', 'default.jpg', '我是Garnett');

/****-----------------------------------------------------------------****/

CREATE TABLE `base_dept`  (
  `dept_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '部门ID',
  `parent_id` bigint(20) NOT NULL COMMENT '上级部门ID',
  `dept_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '部门名称',
  `order_num` double(20, 0) NULL DEFAULT NULL COMMENT '排序',
  `create_time` datetime(0) NULL DEFAULT NULL COMMENT '创建时间',
  `modify_time` datetime(0) NULL DEFAULT NULL COMMENT '修改时间',
  PRIMARY KEY (`dept_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '部门表' ROW_FORMAT = Dynamic;

INSERT INTO `base_dept` VALUES (1, 0, '开发部', 1, '2020-01-04 15:42:26', '2020-01-05 21:08:27');
INSERT INTO `base_dept` VALUES (2, 0, '财务部', 1, '2020-01-04 15:42:26', '2020-01-05 21:08:27');
INSERT INTO `base_dept` VALUES (3, 0, '人事部', 1, '2020-01-04 15:42:26', '2020-01-05 21:08:27');

/****-----------------------------------------------------------------****/

CREATE TABLE `base_role` (
  `role_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '角色ID',
  `role_name` varchar(255) NOT NULL COMMENT '角色名称',
  `remark` varchar(255) DEFAULT NULL COMMENT '角色描述',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `modify_time` datetime DEFAULT NULL COMMENT '修改时间',
  `rev_` int(11) DEFAULT NULL,
  `type_` varchar(255) NOT NULL COMMENT '类型',
  PRIMARY KEY (`role_id`) USING BTREE,
  UNIQUE KEY `role_name` (`role_name`) USING BTREE COMMENT '唯一索引，保证role_name值唯一'
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='角色表';

INSERT INTO `base_role` VALUES (1, 'Admin', 'Admin', '2020-08-08 16:23:11', '2020-08-09 14:38:59',1,'security-role');
INSERT INTO `base_role` VALUES (2, 'cashier', '出纳员', '2020-08-08 16:23:11', '2020-08-09 14:38:59',1,'assignment');
INSERT INTO `base_role` VALUES (3, 'deptLeader', '部门经理', '2020-08-08 16:23:11', '2020-08-09 14:38:59',1,'assignment');
INSERT INTO `base_role` VALUES (4, 'generalManager', '总经理', '2020-08-08 16:23:11', '2020-08-09 14:38:59',1,'assignment');
INSERT INTO `base_role` VALUES (5, 'hr', 'HR管理员', '2020-08-08 16:23:11', '2020-08-09 14:38:59',1,'assignment');
INSERT INTO `base_role` VALUES (6, 'supportCrew', '后勤人员', '2020-08-08 16:23:11', '2020-08-09 14:38:59',1,'assignment');
INSERT INTO `base_role` VALUES (7, 'treasurer', '财务人员', '2020-08-08 16:23:11', '2020-08-09 14:38:59',1,'assignment');
INSERT INTO `base_role` VALUES (8, 'user', 'User', '2020-08-08 16:23:11', '2020-08-09 14:38:59',1,'security-role');


/****-----------------------------------------------------------------****/

CREATE TABLE `base_menu`  (
  `menu_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '菜单/按钮ID',
  `parent_id` bigint(20) NOT NULL COMMENT '上级菜单ID',
  `menu_name` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '菜单/按钮名称',
  `path` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '对应路由path',
  `component` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '对应路由组件component',
  `perms` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '权限标识',
  `icon` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '图标',
  `type` char(2) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '类型 0菜单 1按钮',
  `order_num` double(20, 0) NULL DEFAULT NULL COMMENT '排序',
  `create_time` datetime(0) NOT NULL COMMENT '创建时间',
  `modify_time` datetime(0) NULL DEFAULT NULL COMMENT '修改时间',
  PRIMARY KEY (`menu_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '菜单表' ROW_FORMAT = Dynamic;

insert into `base_menu`(`menu_id`, `parent_id`, `menu_name`, `path`, `component`, `perms`, `icon`, `type`, `order_num`, `create_time`, `modify_time`) values (1, 0, '系统管理', '/system', 'layout', null, 'el-icon-set-up', '0', 1, '2020-12-27 16:39:07', '2020-07-20 16:19:04');
insert into `base_menu`(`menu_id`, `parent_id`, `menu_name`, `path`, `component`, `perms`, `icon`, `type`, `order_num`, `create_time`, `modify_time`) values (2, 0, '系统监控', '/monitor', 'layout', null, 'el-icon-data-line', '0', 2, '2020-12-27 16:45:51', '2020-01-23 06:27:12');
insert into `base_menu`(`menu_id`, `parent_id`, `menu_name`, `path`, `component`, `perms`, `icon`, `type`, `order_num`, `create_time`, `modify_time`) values (3, 1, '用户管理', '/system/user', 'system/user/index', 'user:view', '', '0', 1, '2020-12-27 16:47:13', '2020-01-22 06:45:55');
insert into `base_menu`(`menu_id`, `parent_id`, `menu_name`, `path`, `component`, `perms`, `icon`, `type`, `order_num`, `create_time`, `modify_time`) values (4, 1, '角色管理', '/system/role', 'system/role/index', 'role:view', '', '0', 2, '2020-12-27 16:48:09', '2020-04-25 09:01:12');
insert into `base_menu`(`menu_id`, `parent_id`, `menu_name`, `path`, `component`, `perms`, `icon`, `type`, `order_num`, `create_time`, `modify_time`) values (5, 1, '菜单管理', '/system/menu', 'system/menu/index', 'menu:view', '', '0', 3, '2020-12-27 16:48:57', '2020-04-25 09:01:30');

/****-----------------------------------------------------------------****/

CREATE TABLE `base_user_role`  (
  `user_id` bigint(20) NOT NULL COMMENT '用户ID',
  `role_id` bigint(20) NOT NULL COMMENT '角色ID'
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '用户角色关联表' ROW_FORMAT = Dynamic;

INSERT INTO `base_user_role` VALUES (1, 1);
INSERT INTO `base_user_role` VALUES (2, 5);
INSERT INTO `base_user_role` VALUES (3, 1);
INSERT INTO `base_user_role` VALUES (4, 3);
INSERT INTO `base_user_role` VALUES (5, 5);
INSERT INTO `base_user_role` VALUES (6, 6);
INSERT INTO `base_user_role` VALUES (7, 7);
/****-----------------------------------------------------------------****/

CREATE TABLE `base_role_menu`  (
  `role_id` bigint(20) NOT NULL,
  `menu_id` bigint(20) NOT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '角色菜单关联表' ROW_FORMAT = Dynamic;

INSERT INTO `base_role_menu`(`role_id`, `menu_id`) VALUES (1, 1);
INSERT INTO `base_role_menu`(`role_id`, `menu_id`) VALUES (1, 2);
INSERT INTO `base_role_menu`(`role_id`, `menu_id`) VALUES (1, 3);
INSERT INTO `base_role_menu`(`role_id`, `menu_id`) VALUES (1, 4);
INSERT INTO `base_role_menu`(`role_id`, `menu_id`) VALUES (1, 5);
INSERT INTO `base_role_menu`(`role_id`, `menu_id`) VALUES (3, 1);
INSERT INTO `base_role_menu`(`role_id`, `menu_id`) VALUES (3, 2);
INSERT INTO `base_role_menu`(`role_id`, `menu_id`) VALUES (3, 3);
INSERT INTO `base_role_menu`(`role_id`, `menu_id`) VALUES (3, 4);
INSERT INTO `base_role_menu`(`role_id`, `menu_id`) VALUES (3, 5);

```

上面我们不仅给出了建表语句，而且提供了数据插入SQL。可以发现用户表和角色表插入的数据，和我们activiti用户和用户组中的数据是一样的，稍后我们会让Activcit使用上面的用户体系。到这里，我们的库表就准备完了。

## 2、创建web资源服务工程

我们新建一个新的module来管理用户权限模块，这样也符合微服务的思想，工作流模块和web管理模块分离，松耦合。

### 2.1  新建workflow-web

![image-20201015143522494](/images/activiti6-31/image-20201015143522494.png)

引入POM依赖信息：

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

    <artifactId>workflow-web</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.sxdx</groupId>
            <artifactId>workflow-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>p6spy</groupId>
            <artifactId>p6spy</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-extension</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.30</version>
        </dependency>

        <!-- Mysql驱动包 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <!--阿里数据库连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>

        <!-- pagehelper 分页插件 -->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- spring-boot-devtools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional> <!-- 表示依赖不会传递 -->
        </dependency>

        <!-- swagger2-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>io.swagger</groupId>
                    <artifactId>swagger-models</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-models</artifactId>
            <version>1.5.22</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.13</version>
            <!--使用spring boot2整合 pagehelper-spring-boot-starter必须排除一下依赖
                        因为pagehelper-spring-boot-starter也已经在pom依赖了mybatis与mybatis-spring
                        所以会与mybatis-plus-boot-starter中的mybatis与mybatis-spring发生冲突
                    -->
            <exclusions>
                <exclusion>
                    <groupId>org.mybatis</groupId>
                    <artifactId>mybatis</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.mybatis</groupId>
                    <artifactId>mybatis-spring</artifactId>
                </exclusion>
            </exclusions>
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
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-jwt</artifactId>
        </dependency>

    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.1.1.RELEASE</version>
                <configuration>
                    <fork>true</fork> <!-- 如果没有该配置，devtools不会生效 -->
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.4.2</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
        <finalName>${project.artifactId}</finalName>
    </build>
</project>
```

### 2.2 资源服务器配置

workflow-web也是一个资源服务器，我们将workflow-activiti-rest的资源服务配置类直接粘贴过来：

![image-20201015151318087](/images/activiti6-31/image-20201015151318087.png)

然后修改`ResourceServerConfig`类的资源标识：

![image-20201015183614473](/images/activiti6-31/image-20201015183614473.png)

### 2.3  集成Mybatis-Plus、Swagger、logback

在`src/main/resources/spy.properties`路径下新建spy.properties文件。

```xml
appender=com.p6spy.engine.spy.appender.Slf4JLogger
```

在`src/main/resources/spy.properties`路径下新建 logback-spring.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">

    <springProperty scope="context" name="springAppName" source="spring.application.name"/>

    <property name="log.path" value="logs/workflow-web" />
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

在`com.sxdx.workflow.web.config`下新建`SwaggerConfig`

```java
package com.sxdx.workflow.web.config;

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
                .apis(RequestHandlerSelectors.basePackage("com.sxdx.workflow.web.controller"))
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
                .title("workflow-web ")
                .description("flowable-web模块接口")
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

### 2.4 修改配置文件

修改application.yml如下：

```yaml
spring:
  application:
    name: workflow-web
  autoconfigure:
    exclude: com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceAutoConfigure
  # activiti 模块
  # 解决启动报错：class path resource [processes/] cannot be resolved to URL because it does not exist
  activiti:
    check-process-definitions: false
    # 检测身份信息表是否存在
    #db-identity-used: false
    history-level: full
  datasource:
    dynamic:
      p6spy: true
      #druid配置
      druid:
        #初始化大小,最小,最大
        initial-size: 5
        min-idle: 5
        max-active: 20
        #配置获取连接等待超时的时间
        max-wait: 60000
        #配置间隔多久才进行一次检测,检测需要关闭的空闲连接,单位是毫秒
        time-between-eviction-runs-millis: 60000
        #配置一个连接在池中最小生存的时间,单位是毫秒
        min-evictable-idle-time-millis: 300000
        validation-query: SELECT 1 FROM DUAL
        test-while-idle: true
        test-on-borrow: false
        test-on-return: false
        #打开PSCache,并且指定每个连接上PSCache的大小
        pool-prepared-statements: true
        max-pool-prepared-statement-per-connection-size: 20
        #配置监控统计拦截的filters,去掉后监控界面sql无法统计,'wall'用于防火墙
        filters: stat,wall # 注意这个值和druid原生不一致，默认启动了stat,wall
        filter:
          log4j:
            enabled: true
          wall:
            config:
              # 允许多个语句一起执行
              multi-statement-allow: true
            enabled: true
            db-type: mysql
          stat:
            enabled: true
            db-type: mysql
      # 默认源
      primary: activiti-demo
      datasource:
        activiti-demo:
          url: jdbc:mysql://localhost:3306/activiti-demo?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true
          username: root
          password: gaoyipeng
          driver-class-name: com.mysql.jdbc.Driver
          #slave:
          #url: jdbc:mysql://localhost:3306/slave1?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true
          #username: root
          #password: 123456
          #driver-class-name: com.mysql.jdbc.Driver
          #schema: db/schema.sql


server:
  # 服务器的HTTP端口，默认为80
  port: 8201
  servlet:
    context-path: /
  tomcat:
    # tomcat的URI编码
    uri-encoding: UTF-8
    # tomcat最大线程数，默认为200
    max-threads: 800
    # Tomcat启动初始化的线程数，默认值25
    min-spare-threads: 30

# mybatis-plus相关配置
mybatis-plus:
  # xml扫描，多个目录用逗号或者分号分隔（告诉 Mapper 所对应的 XML 文件位置）
  mapper-locations: classpath:mapper/*.xml
  # 以下配置均有默认值,可以不设置
  global-config:
    banner: false
    db-config:
      #主键类型 AUTO:"数据库ID自增" INPUT:"用户输入ID",ID_WORKER:"全局唯一ID (数字类型唯一ID)", UUID:"全局唯一ID UUID";
      id-type: auto
      #字段策略 IGNORED:"忽略判断"  NOT_NULL:"非 NULL 判断")  NOT_EMPTY:"非空判断"
      field-strategy: NOT_EMPTY
      #数据库类型
      db-type: MYSQL
  configuration:
    jdbc-type-for-null: null
    # 是否开启自动驼峰命名规则映射:从数据库列名到Java属性驼峰命名的类似映射
    map-underscore-to-camel-case: true
    # 如果查询结果中包含空值的列，则 MyBatis 在映射的时候，不会映射这个字段
    call-setters-on-nulls: true
    # 这个配置会将执行的sql打印出来，在开发或测试的时候可以用
    #log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
# 项目相关配置
workflow:
  # 名称
  name: workflow-web
  # 版本
  version: 1.0.0
  # 版权年份
  copyrightYear: 2020
```

我们新建一个测试类：

```java
package com.sxdx.workflow.web.controller;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Api(value="demo1", description="demo1")
@RestController
@RequestMapping("/demo")
@Slf4j
public class DemoController {

    @GetMapping(name = "/r1")
    @ApiOperation(value = "测试1",notes = "测试1")
    public String demo(){
        return "hello";
    }

}

```

启动服务

![image-20201016111622811](/images/activiti6-31/image-20201016111622811.png)

访问http://localhost:8201/swagger-ui.html

![image-20201016111337971](/images/activiti6-31/image-20201016111337971.png)

获取token

![image-20201016111752864](/images/activiti6-31/image-20201016111752864.png)

在swagger中输入token信息

![image-20201016111852612](/images/activiti6-31/image-20201016111852612.png)

访问接口：

![image-20201016111948656](/images/activiti6-31/image-20201016111948656.png)

### 2.5 代码生成器

将`workflow-activiti-rest`工程 `com.sxdx.workflow.activiti.rest.util`目录下的CodeGenerator拷贝到workflow-web工程中，并做一些修改：

```java
package com.sxdx.workflow.web.util;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class CodeGenerator {
    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        final String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/workflow-web/src/main/java");
        gc.setAuthor("Garnett");
        gc.setBaseResultMap(true);
        gc.setBaseColumnList(true);
        // 是否打开输出目录 默认为true
        gc.setOpen(false);
        //是否支持AR模式
        gc.setActiveRecord(true);
        gc.setFileOverride(false);//再次生产是否覆盖原文件
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/activiti-demo?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("gaoyipeng");
        mpg.setDataSource(dsc);

        // 包配置
        final PackageConfig pc = new PackageConfig();
        // pc.setModuleName(scanner("模块名"));
        pc.setParent("com.sxdx.workflow.web");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<FileOutConfig>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/workflow-web/src/main/resources/mapper/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        // 配置自定义输出模板
        // 指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        // strategy.setSuperEntityClass("com.fame.common.BaseEntity");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // strategy.setSuperControllerClass("com.fame.common.BaseController");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        //strategy.setSuperEntityColumns("id");
        // strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix("base");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
}

```

我们使用代码生成器，生成代码：

![image-20201016155939098](/images/activiti6-31/image-20201016155939098.png)

这些代码还需要自定义修改一下，

在启动类上添加`@MapperScan("com.sxdx.workflow.web.mapper")`

```java
@SpringBootApplication
@MapperScan("com.sxdx.workflow.web.mapper")
public class WorkFlowWebApplication {

    public static void main(String[] args) {
        SpringApplication.run(WorkFlowWebApplication.class, args);
    }
}

```

在Mapper接口上添加`@Repository`

![image-20201017225131697](/images/activiti6-31/image-20201017225131697.png)

表和基本代码已经都有了，后期需要什么接口，再补充就行，这里不再赘述。