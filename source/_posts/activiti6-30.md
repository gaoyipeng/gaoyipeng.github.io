---
title: Activiti（30） 数据库存储Code,Client信息
date: 2020-10-05 15:09:49
tags: [Activiti,Spring-Security-OAuth2]
categories: [Activiti,Spring-Security-OAuth2]
description: Activiti（30） 数据库存储Code,Client信息
typora-root-url: ..
---

前面我们我们在集成Oauth2功能的的时候，授权码和客户端信息都是保存在内存中的，但是在生产环境下，这些信息都应该保存在数据库中，否则服务重启后，内存信息都会清空，而且不方便配置。

## 1、数据库表

首先我们需要新建2张表来保存授权码和客户端信息：

```sql
CREATE TABLE `oauth_client_details` (
  `client_id` varchar(255) NOT NULL COMMENT '客户端标识',
  `resource_ids` varchar(255) DEFAULT NULL COMMENT '接入资源列表',
  `client_secret` varchar(255) DEFAULT NULL COMMENT '客户端秘钥',
  `scope` varchar(255) DEFAULT NULL,
  `authorized_grant_types` varchar(255) DEFAULT NULL,
  `web_server_redirect_uri` varchar(255) DEFAULT NULL,
  `authorities` varchar(255) DEFAULT NULL,
  `access_token_validity` int(11) DEFAULT NULL,
  `refresh_token_validity` int(11) DEFAULT NULL,
  `additional_information` longtext,
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `archived` tinyint(4) DEFAULT NULL,
  `trusted` tinyint(4) DEFAULT NULL,
  `autoapprove` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`client_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='接入客户端信息';

CREATE TABLE `oauth_code` (
  `code` varchar(255) DEFAULT NULL,
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `authentication` blob,
  KEY `code_index` (`code`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT;
```

表中插入数据：

```sql
INSERT INTO `activiti-demo`.`oauth_client_details`(`client_id`, `resource_ids`, `client_secret`, `scope`, `authorized_grant_types`, `web_server_redirect_uri`, `authorities`, `access_token_validity`, `refresh_token_validity`, `additional_information`, `create_time`, `archived`, `trusted`, `autoapprove`) VALUES ('activiti-rest', 'activiti-rest,activiti-web,activiti-auth', '$2a$10$RwjayO3IOpr0L8t8We2YC.FoEodRUQJowNPsOOKmkF8hZ4Mfn/V8y', 'ROLE_ADMIN,ROLE_USER,ROLE_API', 'client_credentials,password,authorization_code,implicit,refresh_token', 'https://www.baidu.com', NULL, 7200, 259200, NULL, '2020-09-28 13:50:46', 0, 0, 'true');

INSERT INTO `activiti-demo`.`oauth_client_details`(`client_id`, `resource_ids`, `client_secret`, `scope`, `authorized_grant_types`, `web_server_redirect_uri`, `authorities`, `access_token_validity`, `refresh_token_validity`, `additional_information`, `create_time`, `archived`, `trusted`, `autoapprove`) VALUES ('activiti-web', 'activiti-rest,activiti-web,activiti-auth', '$2a$10$RwjayO3IOpr0L8t8We2YC.FoEodRUQJowNPsOOKmkF8hZ4Mfn/V8y', 'ROLE_ADMIN,ROLE_USER,ROLE_API', 'client_credentials,password,authorization_code,implicit,refresh_token', 'https://www.baidu.com', NULL, 7200, 259200, NULL, '2020-09-28 13:51:00', 0, 0, 'true');

```

![image-20201005195451023](/images/activiti6-30/image-20201005195451023.png)

此处定义了2个客户端信息，其中的`client_secret`字段，明文是123456，此处需要保存加密后的字符串，因为我们代码中使用的是`BCryptPasswordEncoder`加密，所以使用如下方法即可获取加密串：

```java
@Test
public void test1(){
    System.out.println("-------------"+new BCryptPasswordEncoder().encode("123456"));
}
```

## 2、认证服务器集成Mybatis-Plus、Druid

因为涉及到了数据库操作，所以我们需要在认证服务器中集成Mybatis-plus。这个集成步骤在前面搭建workflow-activiti-rest服务时已经介绍过了。

### 2.1 添加依赖

在`workflow-auth`服务的pom中添加：

```xml
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
```

### 2.2 spy.properties

在`src/main/resources/spy.properties`路径下新建spy.properties文件。用于格式化SQL语句。

spy.properties内容如下：

```properties
appender=com.p6spy.engine.spy.appender.Slf4JLogger
```

### 2.3 修改配置

```yaml
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
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    
spring:
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
          password: *******
          driver-class-name: com.mysql.jdbc.Driver
          #slave:
          #url: jdbc:mysql://localhost:3306/slave1?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true
          #username: root
          #password: 123456
          #driver-class-name: com.mysql.jdbc.Driver
          #schema: db/schema.sql
```

### 2.4 Druid配置

将workflow-activiti-rest服务config目录下的DruidConfiguration.java拷贝到workflow-auth的config目录下：

![image-20201005195132971](/images/activiti6-30/image-20201005195132971.png)

## 3、修改资源拦截配置

我们需要修改一下资源配置类`ResourceServerConfig`，将druid、swagger等资源放行。

```java
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
        .antMatchers("/druid/**").permitAll()
        .antMatchers("/**").authenticated();
}
```

![image-20201005215018337](/images/activiti6-30/image-20201005215018337.png)

![image-20201005215030074](/images/activiti6-30/image-20201005215030074.png)

## 4、修改授权服务

### 4.1 修改AuthorizationServerConfig

之前的客户端信息、授权码服务，我们是这样配置的：

```java
/**
     * 配置客户端详情信息
     * @param clients
     * @throws Exception
     */
public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients.inMemory()
        .withClient("kiki")
        .secret(passwordEncoder.encode("123456"))
        .authorizedGrantTypes("password","authorization_code","client_credentials" ,"implicit","refresh_token")
        .autoApprove(true)
        .scopes("all")
        .redirectUris("https://www.baidu.com")
        .resourceIds("activiti-rest","activiti-web","activiti-auth");
}

/**
  * 授权码服务：配置授权码模式下授权码存储方式
  * @return
  */
@Bean
public AuthorizationCodeServices authorizationCodeServices(){
    return new InMemoryAuthorizationCodeServices();
}
```

现在我们修改为：

```java

@Autowired
private ClientDetailsService clientDetailsService;

/**
  * 配置客户端详情信息
  * @param clients
  * @throws Exception
  */
public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients.withClientDetails(clientDetailsService);
}

/**
  * 数据库获取客户端信息
  * @param dataSource
  * @return
  */
@Bean
public ClientDetailsService clientDetailsService(DataSource dataSource) {
    ClientDetailsService clientDetailsService = new JdbcClientDetailsService(dataSource);
    ((JdbcClientDetailsService) clientDetailsService).setPasswordEncoder(passwordEncoder);
    return clientDetailsService;
}

/**
  * 授权码服务：配置授权码模式下授权码存储方式
  * @return
  */
@Bean
public AuthorizationCodeServices authorizationCodeServices(DataSource dataSource){
    return new JdbcAuthorizationCodeServices(dataSource);//设置授权码模式的授权码如何存取
    //return new InMemoryAuthorizationCodeServices();
}
```

这样就修改为通过数据库保存client和code信息了。可以发现，此处我们并没有指定表名，那么不需要告诉程序从那张表获取数据吗，不需要，这是因为Oauth2已经设置了默认的表名和表结构，我们只需创建好对应的表即可。

表结构可以参考github：https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/test/resources/schema.sql

## 5、验证

我们重启服务后验证授权码模式。

浏览器访问：http://localhost:8101/oauth/authorize?response_type=code&client_id=activiti-rest&redirect_uri=https://www.baidu.com&state=state

![image-20201005195328343](/images/activiti6-30/image-20201005195328343.png)

![image-20201005200143937](/images/activiti6-30/image-20201005200143937.png)

此时查看`oauth_code`表：

![image-20201005200229939](/images/activiti6-30/image-20201005200229939.png)

可以看到授权码已经保存到了数据库中。我们使用code获取token：

![image-20201005201246472](/images/activiti6-30/image-20201005201246472.png)

我们还需要设置请求头，key为Authorization，value为Basic加上client_id:client_secret经过base64加密后的值（可以使用http://tool.chinaz.com/Tools/Base64.aspx）

![image-20201005201034760](/images/activiti6-30/image-20201005201034760.png)

![image-20201005201306951](/images/activiti6-30/image-20201005201306951.png)

![image-20201005201329024](/images/activiti6-30/image-20201005201329024.png)

可以看到我们成功获取了token。

此时再次查看`oauth_code`表：

![image-20201005201407526](/images/activiti6-30/image-20201005201407526.png)

发现记录已经被清空了。