---
title: Activiti（10） 普通表单实战,集成Thymeleaf,Mybatis-Plus
date: 2020-06-22 15:55:02
tags: [Activiti,Thymeleaf,Mybatis-Plus]
categories: Activiti
description: Activiti（10） 普通表单实战,集成Thymeleaf,Mybatis-Plus
typora-root-url: ..
password: kiki
---

> 前面我们介绍了动态表单和外置表单。还有一种称为普通表单。
>
> 普通表单的特点是把表单的内容存放在一个页面（jsp、jsf、html等）文件中。表单提供流程所需的各种字段，提交到后台后，流程引擎利用这些字段推动流程。

外置表单解决了动态表单需要代码生成页面的弊端。但是表单业务字段依然保存在Activiti 的`act_hi_detail`表中。无法做到业务数据和流程数据的分离。而普通表单可以解决这个问题。

# 1、Springboot集成 Thymeleaf

因为普通表单会用到页面，所以先来集成Thymeleaf（此处我们使用springboot推荐的Thymeleaf）。

## 1.1 引入jar包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## 1.2 yaml中添加下添加

```yaml
spring:
  thymeleaf:
    cache: false
```



## 1.3 新建templates目录

在resources目录下建templates目录。Thymeleaf会默认到此目录下寻找html文件。

这里贴出一些Thymeleaf默认配置

```properties
#开启模板缓存（默认值：true）
spring.thymeleaf.cache=true 
#Check that the template exists before rendering it.
spring.thymeleaf.check-template=true 
#检查模板位置是否正确（默认值:true）
spring.thymeleaf.check-template-location=true
#Content-Type的值（默认值：text/html）
spring.thymeleaf.content-type=text/html
#开启MVC Thymeleaf视图解析（默认值：true）
spring.thymeleaf.enabled=true
#模板编码
spring.thymeleaf.encoding=UTF-8
#要被排除在解析之外的视图名称列表，用逗号分隔
spring.thymeleaf.excluded-view-names=
#要运用于模板之上的模板模式。另见StandardTemplate-ModeHandlers(默认值：HTML5)
spring.thymeleaf.mode=HTML5
#在构建URL时添加到视图名称前的前缀（默认值：classpath:/templates/）
spring.thymeleaf.prefix=classpath:/templates/
#在构建URL时添加到视图名称后的后缀（默认值：.html）
spring.thymeleaf.suffix=.html
#Thymeleaf模板解析器在解析器链中的顺序。默认情况下，它排第一位。顺序从1开始，只有在定义了额外的TemplateResolver Bean时才需要设置这个属性。
spring.thymeleaf.template-resolver-order=
#可解析的视图名称列表，用逗号分隔
spring.thymeleaf.view-names=
```

一般开发中将`spring.thymeleaf.cache`设置为false，其他保持默认值即可。

## 1.4 测试

- controller：

```java
package com.sxdx.workflow.activiti.rest.controller;

import io.swagger.annotations.Api;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

@Api(value="普通表单模块", description="普通表单模块")
@Controller
@RequestMapping("/normalForm")
@Slf4j
public class NormalFormController {

    @GetMapping (value = "/hello")
    public String index(Model model){
        model.addAttribute("name", "JSP");
        return "hello";
    }
}
```

- templates目录下新建 hello.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>springboot-thymeleaf demo</title>
</head>

<body>
    <p th:text="'hello, ' + ${name} + '!'" />
</body>
</html>
```

- 访问：http://localhost:9090/normalForm/hello

![image-20200624162253595](/images/activiti6-10/image-20200624162253595.png)

# 2、集成Mybatis-Plus，配置多数据源

> Mybatis-Plus（简称MP）是一个 Mybatis 的增强工具，在 Mybatis 的基础上只做增强不做改变，为简化开发、提高效率而生。与Spring的整合亦非常简单。只需把mybatis的依赖换成mybatis-plus的依赖，再把sqlSessionFactory换成mybatis-plus的即可。

我认为所有的项目，一开始搭建时都应该具备多数据源的能力，所以这里我们集成Mybatis-Plus，当然Mybatis-Plus的扩展功能不止这些。Mybatis-Plus官网：https://mp.baomidou.com/

## 2.1 引入jar包

注释掉原先的mybatis依赖。此处我们使用mybatis-plus最新版本：3.3.2

版本说明

```
<mybatisplus.version>3.3.2</mybatisplus.version>
<dynamic.version>3.1.1</dynamic.version>
<p6spy.version>3.8.0</p6spy.version>
<druid.version>1.1.23</druid.version>
```

`mybatis-plus-extension`版本也是3.3.2。这个必须有，否则启动会报`Caused by: java.lang.ClassNotFoundException: org.mybatis.logging.LoggerFactory`

**另外，需要提高Druid的版本到1.1.23，之前的版本与mybatis-plus最新版本不匹配，使用mybatis-plus条件构造器会报错。**

```xml
<!-- <dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
 </dependency>-->

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
```

## 2.2 spy.properties

在`src/main/resources/spy.properties`路径下新建spy.properties文件。用于格式化SQL语句。

```reStructuredText
# 如在使用mybatis的过程中，原生输出的语句是带?号的。在需要复制到其他地方执行看效果的时候很不方便。
select * from user where age>?
# 在使用了p6sy后，其会帮你格式化成真正的执行语句。
select * from user where age>6
```

spy.properties内容如下：

```properties
appender=com.p6spy.engine.spy.appender.Slf4JLogger
```

## 2.3 修改配置

参考：

https://mp.baomidou.com/guide/dynamic-datasource.html

https://github.com/baomidou/dynamic-datasource-spring-boot-starter/wiki

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
    
 # Spring配置
spring:
  autoconfigure:
    exclude: com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceAutoConfigure
  # activiti 模块
  # 解决启动报错：class path resource [processes/] cannot be resolved to URL because it does not exist
  activiti:
    check-process-definitions: false
    # 检测身份信息表是否存在
    #db-identity-used: false
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
          password: ******
          driver-class-name: com.mysql.jdbc.Driver
        slave:
          url: jdbc:mysql://localhost:3306/slave1?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true
          username: root
          password: ******
          driver-class-name: com.mysql.jdbc.Driver
          schema: db/schema.sql
```

**为什么要排除DruidDataSourceAutoConfigure ？**

> DruidDataSourceAutoConfigure会注入一个DataSourceWrapper，其会在原生的spring.datasource下找url,username,password等。而我们动态数据源的配置路径是变化的。

schema.sql：用来初始化数据库的。

```sql
CREATE TABLE IF NOT EXISTS `USER`
(
    id     BIGINT(20) NOT NULL AUTO_INCREMENT,
    name   VARCHAR(30) NULL DEFAULT NULL,
    age    INT(11) NULL DEFAULT NULL,
    PRIMARY KEY (id)
);
```

## 2.4 配置分页

```java
package com.sxdx.workflow.activiti.rest.config;

import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@EnableTransactionManagement
@Configuration
public class MybatisPlusConfig {

    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }

}

```

## 2.5  配置mapper扫描路径

```java
@MapperScan("com.sxdx.workflow.activiti.rest.mapper")
public class WorkFlowActivitiRestApplication {
    public static void main(String[] args)
    {
        SpringApplication.run(WorkFlowActivitiRestApplication.class, args);
        System.out.println("启动成功");
    }
}
```

启动项目，可以正常启动，并且slave1数据库生成了user表。

![image-20200626182643393](/images/activiti6-10/image-20200626182643393.png)



## 2.6  集成代码生成器

Mybatis-Plus帮我们集成了代码生成器。MyBatis-Plus 从 `3.0.3` 之后移除了代码生成器与模板引擎的默认依赖，需要手动添加相关依赖：

**注意版本对应：本地测试时，使用了低版本的`mybatis-plus-generator`和`freemarker`时，代码生成器会报错。**

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.3.2</version>
</dependency>
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.30</version>
</dependency>
```

使用官方提供的代码：

```java
package com.sxdx.workflow.activiti.rest.util;

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
        gc.setOutputDir(projectPath + "/workflow-activiti-rest/src/main/java");
        gc.setAuthor("Garnett");
        gc.setBaseResultMap(true);
        gc.setBaseColumnList(true);
        // 是否打开输出目录 默认为true
        gc.setOpen(false);
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
        pc.setParent("com.sxdx.workflow.activiti.rest");
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
                return projectPath + "/workflow-activiti-rest/src/main/resources/mapper/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
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
        strategy.setTablePrefix("oa");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
}

```

**注意**：要把`strategy.setSuperEntityColumns("id");`注释掉，否则生成的实体类没有id。

然后生成代码：

![image-20200627100305087](/images/activiti6-10/image-20200627100305087.png)

![image-20200627101642117](/images/activiti6-10/image-20200627101642117.png)

# 3、普通表单

## 3.1 流程定义

依然是先贴出流程定义文件。还是之前的请假流程。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.kafeitu.me/activiti/leave">
  <process id="leave" name="请假流程-普通表单" isExecutable="true">
    <documentation>请假流程演示</documentation>
    <startEvent id="startevent1" name="Start" activiti:initiator="applyUserId"></startEvent>
    <userTask id="deptLeaderVerify" name="部门领导审批" activiti:candidateGroups="deptLeader"></userTask>
    <exclusiveGateway id="exclusivegateway5" name="Exclusive Gateway"></exclusiveGateway>
    <userTask id="modifyApply" name="调整申请" activiti:assignee="${applyUserId}"></userTask>
    <userTask id="hrVerify" name="人事审批" activiti:candidateGroups="hr"></userTask>
    <exclusiveGateway id="exclusivegateway6" name="Exclusive Gateway"></exclusiveGateway>
    <userTask id="reportBack" name="销假" activiti:assignee="${applyUserId}">
      <extensionElements>
        <activiti:taskListener event="complete" delegateExpression="${reportBackEndProcessor}"></activiti:taskListener>
      </extensionElements>
    </userTask>
    <endEvent id="endevent1" name="End"></endEvent>
    <exclusiveGateway id="exclusivegateway7" name="Exclusive Gateway"></exclusiveGateway>
    <sequenceFlow id="flow2" sourceRef="startevent1" targetRef="deptLeaderVerify"></sequenceFlow>
    <sequenceFlow id="flow3" sourceRef="deptLeaderVerify" targetRef="exclusivegateway5"></sequenceFlow>
    <sequenceFlow id="flow4" name="不同意" sourceRef="exclusivegateway5" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderApproved == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow5" name="同意" sourceRef="exclusivegateway5" targetRef="hrVerify">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderApproved  == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow6" sourceRef="hrVerify" targetRef="exclusivegateway6"></sequenceFlow>
    <sequenceFlow id="flow7" name="同意" sourceRef="exclusivegateway6" targetRef="reportBack">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${hrApproved  == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow8" sourceRef="reportBack" targetRef="endevent1"></sequenceFlow>
    <sequenceFlow id="flow9" name="不同意" sourceRef="exclusivegateway6" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${!hrApproved  == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow10" name="重新申请" sourceRef="exclusivegateway7" targetRef="deptLeaderVerify">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply  == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow11" sourceRef="modifyApply" targetRef="exclusivegateway7"></sequenceFlow>
    <sequenceFlow id="flow12" name="结束流程" sourceRef="exclusivegateway7" targetRef="endevent1">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${!reApply  == 'false'}]]></conditionExpression>
    </sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_leave">
    <bpmndi:BPMNPlane bpmnElement="leave" id="BPMNPlane_leave">
      <bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="0.0" y="46.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="deptLeaderVerify" id="BPMNShape_deptLeaderVerify">
        <omgdc:Bounds height="55.0" width="105.0" x="80.0" y="36.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway5" id="BPMNShape_exclusivegateway5">
        <omgdc:Bounds height="40.0" width="40.0" x="240.0" y="43.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="modifyApply" id="BPMNShape_modifyApply">
        <omgdc:Bounds height="55.0" width="105.0" x="208.0" y="114.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="hrVerify" id="BPMNShape_hrVerify">
        <omgdc:Bounds height="55.0" width="105.0" x="348.0" y="36.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway6" id="BPMNShape_exclusivegateway6">
        <omgdc:Bounds height="40.0" width="40.0" x="485.0" y="43.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="reportBack" id="BPMNShape_reportBack">
        <omgdc:Bounds height="55.0" width="105.0" x="580.0" y="36.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="615.0" y="195.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway7" id="BPMNShape_exclusivegateway7">
        <omgdc:Bounds height="40.0" width="40.0" x="240.0" y="192.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2">
        <omgdi:waypoint x="35.0" y="63.0"></omgdi:waypoint>
        <omgdi:waypoint x="80.0" y="63.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow3" id="BPMNEdge_flow3">
        <omgdi:waypoint x="185.0" y="63.0"></omgdi:waypoint>
        <omgdi:waypoint x="240.0" y="63.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow4" id="BPMNEdge_flow4">
        <omgdi:waypoint x="260.0" y="83.0"></omgdi:waypoint>
        <omgdi:waypoint x="260.0" y="114.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="33.0" x="270.0" y="83.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow5" id="BPMNEdge_flow5">
        <omgdi:waypoint x="280.0" y="63.0"></omgdi:waypoint>
        <omgdi:waypoint x="348.0" y="63.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="22.0" x="300.0" y="46.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow6" id="BPMNEdge_flow6">
        <omgdi:waypoint x="453.0" y="63.0"></omgdi:waypoint>
        <omgdi:waypoint x="485.0" y="63.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow7" id="BPMNEdge_flow7">
        <omgdi:waypoint x="525.0" y="63.0"></omgdi:waypoint>
        <omgdi:waypoint x="580.0" y="63.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="22.0" x="539.0" y="46.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow8" id="BPMNEdge_flow8">
        <omgdi:waypoint x="632.0" y="91.0"></omgdi:waypoint>
        <omgdi:waypoint x="632.0" y="195.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow9" id="BPMNEdge_flow9">
        <omgdi:waypoint x="505.0" y="83.0"></omgdi:waypoint>
        <omgdi:waypoint x="504.0" y="141.0"></omgdi:waypoint>
        <omgdi:waypoint x="313.0" y="141.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="33.0" x="515.0" y="83.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow10" id="BPMNEdge_flow10">
        <omgdi:waypoint x="240.0" y="212.0"></omgdi:waypoint>
        <omgdi:waypoint x="132.0" y="212.0"></omgdi:waypoint>
        <omgdi:waypoint x="132.0" y="91.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="44.0" x="142.0" y="192.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow11" id="BPMNEdge_flow11">
        <omgdi:waypoint x="260.0" y="169.0"></omgdi:waypoint>
        <omgdi:waypoint x="260.0" y="192.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow12" id="BPMNEdge_flow12">
        <omgdi:waypoint x="280.0" y="212.0"></omgdi:waypoint>
        <omgdi:waypoint x="615.0" y="212.0"></omgdi:waypoint>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="11.0" width="44.0" x="429.0" y="219.0"></omgdc:Bounds>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

## 3.2  业务数据表

```sql
CREATE TABLE `oa_leave` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` varchar(255) DEFAULT NULL COMMENT '用户id',
  `apply_time` datetime DEFAULT NULL COMMENT '申请时间',
  `start_time` datetime DEFAULT NULL COMMENT '开始时间',
  `end_time` datetime DEFAULT NULL COMMENT '结束时间',
  `leave_type` varchar(255) DEFAULT NULL COMMENT '请假类型',
  `process_instance_id` varchar(255) DEFAULT NULL COMMENT '流程实例id',
  `reality_end_time` datetime DEFAULT NULL COMMENT '真实请假结束时间',
  `reality_start_time` datetime DEFAULT NULL COMMENT '真实请假开始时间',
  `reason` varchar(255) DEFAULT NULL COMMENT '请假原因',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



## 3.3 部署流程定义

![image-20200629181542993](/images/activiti6-10/image-20200629181542993.png)

## 3.4 获取启动节点的普通表单

- controller

```java
@Controller
@RequestMapping("/leave")
public class LeaveController {

    @GetMapping(value = {"apply", ""})
    public String createForm(Model model) {
        model.addAttribute("leave", new Leave());
        return "leave/leaveApply";
    }
}

```

- leaveApply.html (集成了layui)

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>请假申请</title>
    <link rel="stylesheet" th:href="@{/layui/css/layui.css}">
</head>
<body>
<fieldset class="layui-elem-field layui-field-title" style="margin-top: 20px;">
    <legend>请假申请</legend>
</fieldset>
<form class="layui-form" th:action="@{/leave/start}" th:object="${leave}" method="post" enctype="application/json">
    <div class="layui-form-item">
        <div class="layui-inline">
            <label class="layui-form-label">请假类型</label>
            <div class="layui-input-inline">
                <select lay-verify="required" lay-search=""  th:id="leaveType" th:field="*{leaveType}">
                    <option value="">直接选择或搜索选择</option>
                    <option value="公休">公休</option>
                    <option value="病假">病假</option>
                    <option value="调休">调休</option>
                    <option value="事假">事假</option>
                    <option value="婚假">婚假</option>
                </select>
            </div>
        </div>
    </div>
    <div class="layui-form-item">
        <div class="layui-inline">
            <label class="layui-form-label">开始时间</label>
            <div class="layui-input-inline">
                <input type="text"  th:id="startTime" th:field="*{startTime}" autocomplete="off"  placeholder="yyyy-MM-dd HH:mm:ss"  class="layui-input">
            </div>
        </div>
    </div>
    <div class="layui-form-item">
        <div class="layui-inline">
            <label class="layui-form-label">结束时间</label>
            <div class="layui-input-inline">
                <input type="text" th:id="endTime" th:field="*{endTime}"  autocomplete="off" placeholder="yyyy-MM-dd HH:mm:ss"  class="layui-input">
            </div>
        </div>
    </div>
    <div class="layui-form-item layui-form-text">
        <label class="layui-form-label">请假原因</label>
        <div class="layui-input-block">
            <textarea placeholder="请输入内容" th:field="*{reason}" class="layui-textarea"></textarea>
        </div>
    </div>
    <div class="layui-form-item">
        <div class="layui-input-block">
            <button type="submit" class="layui-btn" lay-submit="" lay-filter="demo1">立即提交</button>
            <button type="reset" class="layui-btn layui-btn-primary">重置</button>
        </div>
    </div>
</form>

</body>
<script type="text/javascript" th:src="@{/layui/layui.all.js}"></script>
<script>
    layui.use(['form', 'layedit', 'laydate'], function(){
        var form = layui.form
            ,layer = layui.layer
            ,laydate = layui.laydate;

        //日期
        laydate.render({
            elem: '#startTime'
            ,type: 'datetime'
        });
        laydate.render({
            elem: '#endTime'
            ,type: 'datetime'
        });


        //监听提交
        form.on('submit(demo1)', function(data){
            layer.alert(JSON.stringify(data.field), {
                title: '最终的提交信息'
            })
            return true;
        });

    });
</script>
</html>
```

- 访问 http://localhost:9090/leave/apply

![image-20200629173818865](/images/activiti6-10/image-20200629173818865.png)

## 3.5 发起流程

主要代码如下：

```java
@PostMapping(value = {"start"})
@ResponseBody
public CommonResponse startForm(Leave leave) {
    Leave leave1 = leaveService.startForm(leave);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("成功发起普通表单请假流程").data(leave1);
}


 @Transactional
@Override
public Leave startForm(Leave leave) {

    leave.setUserId("kafeitu");
    leave.setApplyTime(LocalDateTime.now());
    //TODO 保存业务数据
    leaveMapper.insert(leave);
    //TODO 发起流程,并建立关联关系
    Map<String, Object> variables = new HashMap<String, Object>();
    ProcessInstance processInstance = normalFormService.startWorkflow(PROCESSDEFINITIONKEY, leave.getId().toString(), variables);
    String processInstanceId = processInstance.getId();
    leave.setProcessInstanceId(processInstanceId);
    leaveMapper.updateById(leave);
    log.debug("start process of {key={}, bkey={}, pid={}, variables={}}", new Object[]{"leave", leave.getId(), processInstanceId, variables});
    return leave;
}

public interface NormalFormService  {
    ProcessInstance startWorkflow(String processDefinitionKey, String businessKey, Map<String, Object> variables);
}

/**
     * 启动流程
     * @param processDefinitionKey 流程定义key
     * @param businessKey 业务数据id
     * @param variables  表单字段，此处为null,因为普通表单字段存放在单独的表中
     * @return
     */
@Override
public ProcessInstance startWorkflow(String processDefinitionKey, String businessKey, Map<String, Object> variables) {

    ProcessInstance processInstance = null;
    try {
        // 用来设置启动流程的人员ID，引擎会自动把用户ID保存到activiti:initiator中
        identityService.setAuthenticatedUserId("kafeitu");

        processInstance = runtimeService.startProcessInstanceByKey("leave", businessKey, variables);
        String processInstanceId = processInstance.getId();

    } finally {
        identityService.setAuthenticatedUserId(null);
    }
    return processInstance;
}
```

注意：需要添加事务控制。防止出现流程数据和业务数据不对应的问题。

![image-20200701100829658](/images/activiti6-10/image-20200701100829658.png)

返回值：

```json
{
    "code":"0000",
    "data":{
        "id":1,
        "userId":"kafeitu",
        "applyTime":"2020-07-01T09:56:49.695",
        "startTime":"2020-07-01T00:00:00",
        "endTime":"2020-07-01T00:00:00",
        "leaveType":"病假",
        "processInstanceId":"5",
        "realityEndTime":null,
        "realityStartTime":null,
        "reason":"阿士大夫撒旦"
    },
    "message":"成功发起普通表单请假流程"
}
```

通过返回的`processInstanceId`，我们可以查看到流程图，说明正常启动了。

![image-20200701101745626](/images/activiti6-10/image-20200701101745626.png)

查看数据库，可以看到双向绑定数据：

![image-20200701175932038](/images/activiti6-10/image-20200701175932038.png)

![image-20200701180000839](/images/activiti6-10/image-20200701180000839.png)

## 3.6 部门领导获取待办

```java
@PostMapping(value = "/getTodoList")
@ApiOperation(value = "获取当前人员待办业务数据",notes = "获取当前人员待办业务数据")
@ResponseBody
public CommonResponse getTodoList(@RequestParam(value = "processDefinitionKey", required = true) @ApiParam("流程定义Id(act_re_procdef表KEY)")String processDefinitionKey,
                                  HttpServletRequest request){
    List<Leave> todoList = leaveService.getTodoList(processDefinitionKey, request);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(todoList);
}


@Transactional(readOnly = true)
@Override
public List<Leave> getTodoList(String processDefinitionKey, HttpServletRequest request) {
    List<Leave> results = new ArrayList<Leave>();

    //TODO 此处应该添加获取当前操作人的代码,先写死用 leaderuser。
    UserQueryImpl user = new UserQueryImpl();
    user = (UserQueryImpl)identityService.createUserQuery().userId("leaderuser");

    List<Task> tasks = new ArrayList<Task>();

    tasks = taskService.createTaskQuery().processDefinitionKey(processDefinitionKey)
        .taskCandidateOrAssigned(user.getId()).active().orderByTaskId().desc().list();

    // 根据流程的业务ID查询实体并关联
    for (Task task : tasks) {
        String processInstanceId = task.getProcessInstanceId();
        ProcessInstance processInstance = runtimeService.createProcessInstanceQuery().processInstanceId(processInstanceId).active().singleResult();
        ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery().processDefinitionId(processInstance.getProcessDefinitionId()).singleResult();

        if (processInstance == null) {
            continue;
        }
        String businessKey = processInstance.getBusinessKey();
        if (businessKey == null) {
            continue;
        }
        Leave leave = leaveMapper.selectById(new Long(businessKey));
        leave.setTask(BeanUtil.beanToMap(task));
        leave.setProcessInstance(BeanUtil.beanToMap(processInstance));
        leave.setProcessDefinition(BeanUtil.beanToMap(processDefinition));
        results.add(leave);
    }
    return results;
}
```

此处我们集成了Hutool。用于将Task等对象转为Map。否则会报懒加载无法json序列化的错误。

```xml
<hutool.version>5.3.7</hutool.version>
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>${hutool.version}</version>
</dependency>
```

![image-20200701173805333](/images/activiti6-10/image-20200701173805333.png)

返回信息：

**包含了业务信息已经流程定义，流程实例、流程任务信息。**

```json
{
  "code": "0000",
  "data": [
    {
      "task": {
        "owner": null,
        "originalAssignee": null,
        "assignee": null,
        "delegationState": null,
        "parentTaskId": null,
        "name": "部门领导审批",
        "localizedName": null,
        "description": null,
        "localizedDescription": null,
        "priority": 50,
        "createTime": "2020-07-01 02:38:28",
        "dueDate": null,
        "suspensionState": 1,
        "category": null,
        "executionId": "8",
        "processInstanceId": "5",
        "processDefinitionId": "leave:1:4",
        "taskDefinitionKey": "deptLeaderVerify",
        "formKey": null,
        "isDeleted": false,
        "isCanceled": false,
        "eventName": null,
        "currentActivitiListener": null,
        "tenantId": "",
        "queryVariables": null,
        "claimTime": null,
        "usedVariablesCache": {},
        "cachedElContext": null,
        "id": "11",
        "revision": 1,
        "isInserted": false,
        "isUpdated": false
      },
      "variables": null,
      "processInstance": {
        "currentActivitiListener": null,
        "parent": null,
        "superExecution": null,
        "tenantId": "",
        "name": null,
        "description": null,
        "localizedName": null,
        "localizedDescription": null,
        "lockTime": null,
        "isActive": true,
        "isScope": true,
        "isConcurrent": false,
        "isEnded": false,
        "isEventScope": false,
        "isMultiInstanceRoot": false,
        "isCountEnabled": false,
        "eventName": null,
        "deleteReason": null,
        "suspensionState": 1,
        "startUserId": "kafeitu",
        "startTime": "2020-07-01 02:38:28",
        "eventSubscriptionCount": 0,
        "taskCount": 0,
        "jobCount": 0,
        "timerJobCount": 0,
        "suspendedJobCount": 0,
        "deadLetterJobCount": 0,
        "variableCount": 0,
        "identityLinkCount": 0,
        "processDefinitionId": "leave:1:4",
        "processDefinitionKey": "leave",
        "processDefinitionName": "请假流程-普通表单",
        "processDefinitionVersion": 1,
        "deploymentId": "1",
        "activityId": null,
        "activityName": null,
        "processInstanceId": "5",
        "businessKey": "1",
        "parentId": null,
        "superExecutionId": null,
        "rootProcessInstanceId": "5",
        "rootProcessInstance": null,
        "queryVariables": null,
        "isDeleted": false,
        "usedVariablesCache": {},
        "cachedElContext": null,
        "id": "5",
        "revision": 1,
        "isInserted": false,
        "isUpdated": false
      },
      "historicProcessInstance": null,
      "processDefinition": {
        "name": "请假流程-普通表单",
        "description": "",
        "key": "leave",
        "version": 1,
        "category": "http://www.kafeitu.me/activiti/leave",
        "deploymentId": "1",
        "resourceName": "D:/workflow/uploadPath/processDefiniton/2020/07/01/c25564127e3883b1d360e8cfbd1cf6cb.bpmn",
        "tenantId": "",
        "historyLevel": null,
        "diagramResourceName": "D:/workflow/uploadPath/processDefiniton/2020/07/01/c25564127e3883b1d360e8cfbd1cf6cb.leave.png",
        "isGraphicalNotationDefined": true,
        "variables": null,
        "hasStartFormKey": false,
        "suspensionState": 1,
        "ioSpecification": null,
        "engineVersion": null,
        "id": "leave:1:4",
        "revision": 1,
        "isInserted": false,
        "isUpdated": false,
        "isDeleted": false
      },
      "id": 1,
      "userId": "kafeitu",
      "applyTime": "2020-07-01T10:38:21",
      "startTime": "2020-07-01T00:00:00",
      "endTime": "2020-07-01T00:00:00",
      "leaveType": "病假",
      "processInstanceId": "5",
      "realityEndTime": null,
      "realityStartTime": null,
      "reason": "阿士大夫撒旦"
    }
  ]
}
```

## 3.7 部门领导签收任务

上一步可以获取到taskId为11。

![image-20200701174509977](/images/activiti6-10/image-20200701174509977.png)

## 3.8 部门领导办理任务

这一步其实应该是在审批页面上完成的。步骤3.6 我们已经读取到了表单和流程信息。根据这些信息，生成审批页面即可。

这里懒得写页面了（其实是因为我是后端狗，写页面感觉贼烦，前后端分离好香~~）。注意此处需要把审批人改 为`leaderuser`

我们直接审批通过。

![image-20200701191021431](/images/activiti6-10/image-20200701191021431.png)

此时我们再次查看流程图

![image-20200701191105845](/images/activiti6-10/image-20200701191105845.png)

## 3.9  剩余步骤

剩下的审批步骤和部门经理审批类似，不再详细介绍，注意每个节点表单信息是不一样的。



# 4、总结

普通表单

优点：解决了动态表单需要代码生成页面的弊端。也解决了流程业务数据全部保存在Activiti的表（act_hi_detail）中的弊端，实现了数据解耦。

弊端：灵活，但是繁琐，通用性很差，所有业务都需要单独编码。

# 5、表单模式选型

​		我们已经学习了动态表单、外置表单、普通表单。它们各有利弊。但是设计流程最重要的是“通用”，选择的表单需要能够支持不同的业务系统。

​		综合考虑，**外置表单是最合适的**，我们前面介绍外置表单时，通过`activiti:formKey`设置外置表单。并且部署时需要将`.form`文件和`.bpmn`一起部署。这样是很麻烦的，后期修改能要了程序员的老命。

​		**解决方案**：`activiti:formKey`的值不再是表单文件（.form），而是一个URL，这个URL返回一个表单页面。在办理Task时，我们只需通过任务id获取Form key.

```java
TaskFormData formData = formService.getTaskFormData(taskId);;
String formKey = formData.getFormKey();
```

Form key的主要是获取任务节点需要展示的页面，当我们要打开任务表单的时候可以转发或重定向到任务表单，如：　　

```java
return "redirect:/"+formKey+ "?id="+" +objId + "&taskId=" +taskId";//objId为业务对象Id  taskId为任务Id 这样就可以在任务表单获取到想要的信息
```

