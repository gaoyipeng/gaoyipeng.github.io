---
title: Activiti（8） 动态表单实战、集成Swagger、Logback
date: 2020-06-08 16:51:23
tags: [Activiti,Swagger,logback]
categories: [Activiti,Swagger,logback]
description: Activiti（8） 动态表单实战、集成Swagger、Logback
typora-root-url: ..
password: kiki
---

> 动态表单是直接将工作流节点处的表单嵌入流程定义文件BPMN中。系统利用JS或者模板引擎，根据流程定义中表单定义的各个子控件及其属性动态渲染出表单并加载出来。

在正式开始介绍表单前，我们先完善一下系统。

# 1、工程完善

## 1.1、集成Swagger

在开始实战前，我们先集成Swagger,这样有利于我们后期管理接口。

- 在workflow-activiti-rest module的pom.xml文件中添加：

```xml
 <!-- swagger2-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
```

- 添加配置类

```java
package com.sxdx.workflow.activiti.rest.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    /**
     * 通过 createRestApi函数来构建一个DocketBean
     * 函数名,可以随意命名,喜欢什么命名就什么命名
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())//调用apiInfo方法,创建一个ApiInfo实例,里面是展示在文档页面信息内容
                .select()
                //控制暴露出去的路径下的实例
                //如果某个接口不想暴露,可以使用以下注解
                //@ApiIgnore 这样,该接口就不会暴露在 swagger2 的页面下
                .apis(RequestHandlerSelectors.basePackage("com.sxdx.workflow.activiti.rest"))
                .paths(PathSelectors.any())
                .build();
    }
    //构建 api文档的详细信息函数
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("Springboot2集成Activiti6")
                .contact("Garnett")
                .version("1.0")
                //描述
                .description("Springboot2集成Activiti6")
                .build();
    }
}

```

按swagger的语法规则在`Controller`上添加对应的注解。

**参考**：[在 Spring Boot 项目中使用 Swagger 文档](https://www.ibm.com/developerworks/cn/java/j-using-swagger-in-a-spring-boot-project/index.html)



- 访问 http://localhost:8080/swagger-ui.html

![image-20200608191250469](/images/activiti/activiti6-08/image-20200608191250469.png)

集成Swagger就是这么简单。

## 1.2、安装lombok

不解释lombok 是啥玩意儿，直接贴出集成步骤。

`workflow-activiti `的pom.xml:

```xml
<lombok.version>1.16.10</lombok.version> 
<!--lombok-->
 <dependency>
 	<groupId>org.projectlombok</groupId>
 	<artifactId>lombok</artifactId>
 	<version>${lombok.version}</version>
 </dependency>
```

`workflow-activiti-rest` 的 pom.xml:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

## 1.3、集成logback日志

无需再次引入Jar包，因为springboot本身就内置了日志功能。**在spring-boot-starters**中即可看到：

![image-20200609163952858](/images/activiti/activiti6-08/image-20200609163952858.png)

我们只需要创建`logback-spring.xml`文件即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <contextName>logback</contextName>
    <property resource="application.yml"/>

    <springProperty scope="context" name="log.path" source="logging.path"/>
    <springProperty scope="context" name="log.lv" source="logging.lv"/>
    <springProperty scope="context" name="log.dateSize" source="logging.dateSize"/>

    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>${log.lv}</level>
        </filter>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <!--编码-->
            <charset>utf-8</charset>
        </encoder>
    </appender>

    <!--输出到debug-->
    <appender name="debug" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${log.path}/logback-debug.log</file>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/logback-debug-%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>${log.dateSize}</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
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
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${log.path}/logback-info.log</file>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/logback-info-%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>${log.dateSize}</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
            <!--编码-->
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
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${log.path}/logback-warn.log</file>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/logback-warn-%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>${log.dateSize}</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
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
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${log.path}/logback-error.log</file>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/logback-error-%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>${log.dateSize}</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
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



application中添加日志设置：

```yaml
#日志配置
logging:
  path: E:/Data/idea-workspace/workflow-activiti/logs #保存日志
  lv: info # 控制台日志输出级别
  dateSize: 1 # 日志保存天数
```



![image-20200609164452285](/images/activiti/activiti6-08/image-20200609164452285.png)

# 2、动态表单

接下里我们用一个请假流程来演示，流程定义如下：leave-dynamic-from.bpmn

```xml
<?xml version='1.0' encoding='UTF-8'?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.kafeitu.me/demo/activiti/leave">
  <process id="leave-dynamic-from" name="请假流程-动态表单" isExecutable="true">
    <documentation>请假流程演示-动态表单</documentation>
    <startEvent id="startevent1" name="Start" activiti:initiator="applyUserId">
      <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" required="true"/>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" required="true"/>
        <activiti:formProperty id="reason" name="请假原因" type="string" required="true"/>
      </extensionElements>
    </startEvent>
    <userTask id="deptLeaderAudit" name="部门领导审批" activiti:candidateGroups="deptLeader">
      <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" writable="false"/>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" writable="false"/>
        <activiti:formProperty id="reason" name="请假原因" type="string" writable="false"/>
        <activiti:formProperty id="deptLeaderPass" name="审批意见" type="enum" required="true">
          <activiti:value id="true" name="同意"/>
          <activiti:value id="false" name="不同意"/>
        </activiti:formProperty>
      </extensionElements>
    </userTask>
    <exclusiveGateway id="exclusivegateway5" name="Exclusive Gateway"/>
    <userTask id="modifyApply" name="调整申请" activiti:assignee="${applyUserId}">
      <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" required="true"/>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" required="true"/>
        <activiti:formProperty id="reason" name="请假原因" type="string" required="true"/>
        <activiti:formProperty id="reApply" name="重新申请" type="enum" required="true">
          <activiti:value id="true" name="重新申请"/>
          <activiti:value id="false" name="取消申请"/>
        </activiti:formProperty>
      </extensionElements>
    </userTask>
    <userTask id="hrAudit" name="人事审批" activiti:candidateGroups="hr">
      <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" writable="false"/>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" writable="false"/>
        <activiti:formProperty id="reason" name="请假原因" type="string" writable="false"/>
        <activiti:formProperty id="hrPass" name="审批意见" type="enum" required="true">
          <activiti:value id="true" name="同意"/>
          <activiti:value id="false" name="不同意"/>
        </activiti:formProperty>
      </extensionElements>
    </userTask>
    <exclusiveGateway id="exclusivegateway6" name="Exclusive Gateway"/>
    <userTask id="reportBack" name="销假" activiti:assignee="${applyUserId}">
      <extensionElements>
        <activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" writable="false"/>
        <activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" writable="false"/>
        <activiti:formProperty id="reason" name="请假原因" type="string" writable="false"/>
        <activiti:formProperty id="reportBackDate" name="销假日期" type="date" datePattern="yyyy-MM-dd" required="true"/>
      </extensionElements>
    </userTask>
    <endEvent id="endevent1" name="End"/>
    <exclusiveGateway id="exclusivegateway7" name="Exclusive Gateway"/>
    <sequenceFlow id="flow2" sourceRef="startevent1" targetRef="deptLeaderAudit"/>
    <sequenceFlow id="flow3" sourceRef="deptLeaderAudit" targetRef="exclusivegateway5"/>
    <sequenceFlow id="flow4" name="不同意" sourceRef="exclusivegateway5" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderPass == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow5" name="同意" sourceRef="exclusivegateway5" targetRef="hrAudit">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${deptLeaderPass == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow6" sourceRef="hrAudit" targetRef="exclusivegateway6"/>
    <sequenceFlow id="flow7" name="同意" sourceRef="exclusivegateway6" targetRef="reportBack">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${hrPass == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow8" sourceRef="reportBack" targetRef="endevent1"/>
    <sequenceFlow id="flow9" name="不同意" sourceRef="exclusivegateway6" targetRef="modifyApply">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${hrPass == 'false'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow10" name="重新申请" sourceRef="exclusivegateway7" targetRef="deptLeaderAudit">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'true'}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="flow11" sourceRef="modifyApply" targetRef="exclusivegateway7"/>
    <sequenceFlow id="flow12" name="结束流程" sourceRef="exclusivegateway7" targetRef="endevent1">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${reApply == 'false'}]]></conditionExpression>
    </sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_leave-dynamic-from">
    <bpmndi:BPMNPlane bpmnElement="leave-dynamic-from" id="BPMNPlane_leave-dynamic-from">
      <bpmndi:BPMNShape bpmnElement="startevent1" id="BPMNShape_startevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="10.0" y="90.0"/>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="deptLeaderAudit" id="BPMNShape_deptLeaderAudit">
        <omgdc:Bounds height="55.0" width="105.0" x="90.0" y="80.0"/>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway5" id="BPMNShape_exclusivegateway5">
        <omgdc:Bounds height="40.0" width="40.0" x="250.0" y="87.0"/>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="modifyApply" id="BPMNShape_modifyApply">
        <omgdc:Bounds height="55.0" width="105.0" x="218.0" y="190.0"/>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="hrAudit" id="BPMNShape_hrAudit">
        <omgdc:Bounds height="55.0" width="105.0" x="358.0" y="80.0"/>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway6" id="BPMNShape_exclusivegateway6">
        <omgdc:Bounds height="40.0" width="40.0" x="495.0" y="87.0"/>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="reportBack" id="BPMNShape_reportBack">
        <omgdc:Bounds height="55.0" width="105.0" x="590.0" y="80.0"/>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="endevent1" id="BPMNShape_endevent1">
        <omgdc:Bounds height="35.0" width="35.0" x="625.0" y="283.0"/>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="exclusivegateway7" id="BPMNShape_exclusivegateway7">
        <omgdc:Bounds height="40.0" width="40.0" x="250.0" y="280.0"/>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2">
        <omgdi:waypoint x="45.0" y="107.5"/>
        <omgdi:waypoint x="90.0" y="107.5"/>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow3" id="BPMNEdge_flow3">
        <omgdi:waypoint x="195.0" y="107.29411764705883"/>
        <omgdi:waypoint x="250.078125" y="107.078125"/>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow4" id="BPMNEdge_flow4">
        <omgdi:waypoint x="270.0900900900901" y="126.90990990990991"/>
        <omgdi:waypoint x="270.3755656108597" y="190.0"/>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow5" id="BPMNEdge_flow5">
        <omgdi:waypoint x="289.92907801418437" y="107.0709219858156"/>
        <omgdi:waypoint x="358.0" y="107.31316725978647"/>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow6" id="BPMNEdge_flow6">
        <omgdi:waypoint x="463.0" y="107.2488038277512"/>
        <omgdi:waypoint x="495.0952380952381" y="107.0952380952381"/>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow7" id="BPMNEdge_flow7">
        <omgdi:waypoint x="534.921875" y="107.078125"/>
        <omgdi:waypoint x="590.0" y="107.29411764705883"/>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow10" id="BPMNEdge_flow10">
        <omgdi:waypoint x="250.15503875968992" y="299.84496124031006"/>
        <omgdi:waypoint x="142.0" y="299.0"/>
        <omgdi:waypoint x="142.42819843342036" y="135.0"/>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow8" id="BPMNEdge_flow8">
        <omgdi:waypoint x="642.5" y="135.0"/>
        <omgdi:waypoint x="642.5" y="283.0"/>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow11" id="BPMNEdge_flow11">
        <omgdi:waypoint x="270.3333333333333" y="245.0"/>
        <omgdi:waypoint x="270.12048192771084" y="280.12048192771084"/>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow9" id="BPMNEdge_flow9">
        <omgdi:waypoint x="514.8198198198198" y="126.81981981981981"/>
        <omgdi:waypoint x="514.0" y="217.0"/>
        <omgdi:waypoint x="323.0" y="217.39219712525667"/>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="flow12" id="BPMNEdge_flow12">
        <omgdi:waypoint x="289.97319034852546" y="300.02680965147454"/>
        <omgdi:waypoint x="625.0000157650343" y="300.47651008827523"/>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

![image-20200609094606898](/images/activiti/activiti6-08/image-20200609094606898.png)

可以看到，流程图和之前的请假流程一样，只是在各个节点添加了动态表单。

![image-20200609094620979](/images/activiti/activiti6-08/image-20200609094620979.png)

## 2.1、 初始化流程审批人员

可以参考第4节的单元测试代码来初始化数据，（我是直接套用了kft-activiti-demo项目的表数据）最终表结构如下：

![image-20200609191006625](/images/activiti/activiti6-08/image-20200609191006625.png)

![image-20200609191024047](/images/activiti/activiti6-08/image-20200609191024047.png)

![image-20200609191044731](/images/activiti/activiti6-08/image-20200609191044731.png)



## 2.2 、部署流程定义

直接使用`swagger`上传`.bpmn`文件来部署流程定义。说明（因为本地端口占用，我把端口改为9090了）

![image-20200611094537384](/images/activiti/activiti6-08/image-20200611094537384.png)

![image-20200611094714776](/images/activiti/activiti6-08/image-20200611094714776.png)



可以看到，数据库中生成了一条流程定义记录。

## 2.3、 获取流程启动节点表单信息

我们针对动态表单，新建一个`dynamicFromController.java`文件，动态表单专有的方法写这里：

```java
@Api(value="动态表单模块", description="动态表单模块")
@RestController
@Slf4j
public class dynamicFromController {

    @Autowired
    private FormService formService;

    /**
     * 读取启动流程的表单字段来渲染start form
     */
    @GetMapping(value = "/get-form/start/{reProcdef}")
    @ApiOperation(value = "获取流程启动节点表单信息",notes = "获取流程启动节点表单信息")
    public CommonResponse findStartForm(@PathVariable("reProcdef") @ApiParam("流程定义Id(act_re_procdef表id)") String processDefinitionId) throws Exception {
        Map<String, Object> result = new HashMap<String, Object>();
        StartFormDataImpl startFormData = (StartFormDataImpl) formService.getStartFormData(processDefinitionId);
        startFormData.setProcessDefinition(null);
        /*
         * 读取enum类型数据，用于下拉框
         */
        List<FormProperty> formProperties = startFormData.getFormProperties();
        for (FormProperty formProperty : formProperties) {
            Map<String, String> values = (Map<String, String>) formProperty.getType().getInformation("values");
            if (values != null) {
                for (Map.Entry<String, String> enumEntry : values.entrySet()) {
                    log.debug("enum, key: {}, value: {}", enumEntry.getKey(), enumEntry.getValue());
                }
                result.put("enum_" + formProperty.getId(), values);
            }
        }
        result.put("form", startFormData);
        return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("获取启动节点").data(result);
    }
}

```

测试：

![image-20200611094938822](/images/activiti/activiti6-08/image-20200611094938822.png)

```json
{
  "code": "0000",
  "data": {
    "form": {
      "formKey": null,
      "deploymentId": "3",
      "formProperties": [
        {
          "id": "startDate",
          "name": "请假开始日期",
          "type": {
            "name": "date"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": true
        },
        {
          "id": "endDate",
          "name": "请假结束日期",
          "type": {
            "name": "date"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": true
        },
        {
          "id": "reason",
          "name": "请假原因",
          "type": {
            "name": "string",
            "mimeType": "text/plain"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": true
        }
      ],
      "processDefinition": null
    }
  },
  "message": "获取启动节点"
}
```



可以看到我们已经获取到了start节点的动态表单信息。我们可以根据这些信息，来动态生成一个表单页面。

## 2.4 、提交启动流程

上一步我们已经获取到了启动流程的开始节点所需的表单信息。接下来我们将这些字段赋值后提交到后台，并启动流程。

- Controller层代码

```java
    /**
     * 提交启动流程
     */
    @PostMapping(value = "/start-process/{processDefinitionId}")
    @ApiOperation(value = "提交启动流程",notes = "获取动态表单提交的数据，并发起流程，表单数据key-value结构，key需要以fp_开头")
    public CommonResponse submitStartFormAndStartProcessInstance(@PathVariable("processDefinitionId") @ApiParam("流程定义Id(act_re_procdef表id)")String processDefinitionId,
                                                         HttpServletRequest request) throws CommonException {
        ProcessInstance processInstance = dynamicFromService.submitStartFormAndStartProcessInstance(processDefinitionId,request);
        return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("流程启动成功，流程ID：" + processInstance.getId()).data(processInstance.getId());
    }
```

- Service层代码：

```java
/**
     * 获取动态表单提交的数据，并发起流程，表单数据 key-value结构，key需要以fp_开头
     * @param processDefinitionId
     * @param request
     * @return
     */
    @Override
    @Transactional
    public ProcessInstance submitStartFormAndStartProcessInstance(String processDefinitionId,HttpServletRequest request) throws CommonException {
        Map<String, String> formProperties = new HashMap<String, String>();
        // 从request中读取参数然后转换
        Map<String, String[]> parameterMap = request.getParameterMap();
        if (parameterMap.size()>0){
            Set<Map.Entry<String, String[]>> entrySet = parameterMap.entrySet();
            for (Map.Entry<String, String[]> entry : entrySet) {
                String key = entry.getKey();
                // fp_的意思是form paremeter
                if (StringUtils.defaultString(key).startsWith("fp_")) {
                    formProperties.put(key.split("_")[1], entry.getValue()[0]);
                }
            }
        }else {
            throw new CommonException("表单数据为空");
        }

        log.debug("start form parameters: {}", formProperties);

        //TODO 此处应该添加获取当前操作人的代码,先写死用kafeitu发起流程。用户未登录不能操作，实际应用使用权限框架实现，例如Spring Security、Shiro等
        UserQueryImpl user = new UserQueryImpl();
        user = (UserQueryImpl)identityService.createUserQuery().userId("kafeitu");

        ProcessInstance processInstance = null;
        try {
            identityService.setAuthenticatedUserId(user.getId());
            processInstance = formService.submitStartFormData(processDefinitionId, formProperties);
            log.debug("start a processinstance: {}", processInstance);
        } finally {
            identityService.setAuthenticatedUserId(null);
        }
        return processInstance;
    }
```

因为需要使用form表单方式上传表单数据，不能直接使用swagger。Postman测试如下：

![image-20200611095115978](/images/activiti/activiti6-08/image-20200611095115978.png)

**注意：在字段前加`fp_`前缀，这样来区分表单字段和业务字段。加`fp_`的字段会被添加到工作流数据表中**

此时我们调用**获取流程图,输出跟踪流程信息**，可以看到流程已经到达**部门领导审批**环节。

![image-20200611100619751](/images/activiti/activiti6-08/image-20200611100619751.png)

## 2.5、 获取待办流程列表

流程启动后，下一步是部门经理审批，我们获取**部门经理待办列表**。

- Controller

  ```java
   @PostMapping(value = "/task/list")
      @ResponseBody
      @ApiOperation(value = "获取当前人员待办列表",notes = "获取当前人员待办列表,如果要查询所有，则传all")
      public CommonResponse taskList(@RequestParam(value = "processDefinitionKey", required = true) @ApiParam("流程定义Id(act_re_procdef表KEY)")String processDefinitionKey, HttpServletRequest request) {
          List<Task> taskList = processService.taskList(processDefinitionKey, request);
  
          List<Map<String, Object>> customTaskList = new ArrayList<>();
          for (Task task : taskList) {
              Map<String, Object> map = new LinkedHashMap<>();
              map.put("taskId", task.getId());//任务ID
              map.put("taskName",task.getName());//任务名称
              map.put("taskDefinitionKey", task.getTaskDefinitionKey());//任务Key
              map.put("processDefinitionId", task.getName());//流程定义ID
              map.put("processInstanceId", task.getProcessInstanceId());//	流程实例ID
              map.put("priority", task.getPriority());//	优先级
              map.put("createTime", task.getCreateTime());//	任务创建日期
              map.put("dueDate", task.getDueDate());//任务逾期日
              map.put("description", task.getDescription());//任务描述
              map.put("owner", task.getOwner());
              map.put("assignee", task.getAssignee());//任务所属人（为空需要签收）
              customTaskList.add(map);
          }
          return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("获取当前人员待办列表成功").data(customTaskList);
      }
  ```

  

- Service

  ```java
  @Override
      public List<Task> taskList(String processDefinitionKey, HttpServletRequest request) {
  
          //TODO 此处应该添加获取当前操作人的代码,先写死用 leaderuser。
          UserQueryImpl user = new UserQueryImpl();
          user = (UserQueryImpl)identityService.createUserQuery().userId("leaderuser");
  
          List<Task> tasks = new ArrayList<Task>();
  
          if (!StringUtils.equals(processDefinitionKey, "all")) {
              tasks = taskService.createTaskQuery().processDefinitionKey(processDefinitionKey)
                      .taskCandidateOrAssigned(user.getId()).active().orderByTaskId().desc().list();
          } else {
              tasks = taskService.createTaskQuery().taskCandidateOrAssigned(user.getId()).active().orderByTaskId().desc().list();
          }
          return tasks;
      }
  ```

- 测试

  ![image-20200611095231773](/images/activiti/activiti6-08/image-20200611095231773.png)

##  2.6 、签收任务

上面我们获取了`leaderuser`的待办任务，其中 assignee 为 `null`。说明这个任务需要签收。签收后此字段会变为`leaderuser`

```java
 /**
  * 签收任务
  */
@GetMapping(value = "/task/claim/{id}")
@ResponseBody
@ApiOperation(value = "签收任务",notes = "签收任务")
public CommonResponse claim(@PathVariable("id") @ApiParam("任务id")String taskId, HttpServletRequest request) {
    processService.claim(taskId, request);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("已签收").data("");
}

 @Override
public void claim(String taskId, HttpServletRequest request) {
    //TODO 此处应该添加获取当前操作人的代码,先写死用 leaderuser。
    UserQueryImpl user = new UserQueryImpl();
    user = (UserQueryImpl)identityService.createUserQuery().userId("leaderuser");

    taskService.claim(taskId, user.getId());
}
```

测试：

![image-20200611095518425](/images/activiti/activiti6-08/image-20200611095518425.png)

##  2.7 、`leaderuser` 读取task的表单信息

```java
/**
     * 读取Task的表单
     */
    @GetMapping(value = "get-form/task/{taskId}")
    @ResponseBody
    @ApiOperation(value = "读取task的表单信息",notes = "读取task的表单信息")
    public CommonResponse getTaskFormByTaskId(@PathVariable("taskId") String taskId) throws Exception {
        Map<String, Object> result =  processService.getTaskFormByTaskId(taskId);
        return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("读取Task的表单成功").data(result);
    }


@Override
public Map<String, Object> getTaskFormByTaskId(String taskId) {
    Map<String, Object> result = new HashMap<String, Object>();
    TaskFormDataImpl taskFormData = (TaskFormDataImpl) formService.getTaskFormData(taskId);

    // 设置task为null，否则输出json的时候会报错
    taskFormData.setTask(null);

    result.put("taskFormData", taskFormData);
    /*
         * 读取enum类型数据，用于下拉框
         */
    List<FormProperty> formProperties = taskFormData.getFormProperties();
    for (FormProperty formProperty : formProperties) {
        Map<String, String> values = (Map<String, String>) formProperty.getType().getInformation("values");
        if (values != null) {
            for (Map.Entry<String, String> enumEntry : values.entrySet()) {
                log.debug("enum, key: {}, value: {}", enumEntry.getKey(), enumEntry.getValue());
            }
            result.put(formProperty.getId(), values);
        }
    }
    return result;
}

```

swagger测试：

![image-20200611095658107](/images/activiti/activiti6-08/image-20200611095658107.png)

我们贴出返回值,可以看到发起人填写的表单信息，部门经理节点是可以看到的。这是因为表单的字段信息都被流程引擎保存在`act_hi_detail`表中了。

![image-20200617113356897](/images/activiti6-08/image-20200617113356897.png)

```json
{
  "code": "0000",
  "data": {
    "taskFormData": {
      "formKey": null,
      "deploymentId": "3",
      "formProperties": [
        {
          "id": "startDate",
          "name": "请假开始日期",
          "type": {
            "name": "date"
          },
          "value": "2020-02-03",
          "readable": true,
          "writable": false,
          "required": false
        },
        {
          "id": "endDate",
          "name": "请假结束日期",
          "type": {
            "name": "date"
          },
          "value": "2020-02-04",
          "readable": true,
          "writable": false,
          "required": false
        },
        {
          "id": "reason",
          "name": "请假原因",
          "type": {
            "name": "string",
            "mimeType": "text/plain"
          },
          "value": "世界这么大，我想去看看",
          "readable": true,
          "writable": false,
          "required": false
        },
        {
          "id": "deptLeaderPass",
          "name": "审批意见",
          "type": {
            "name": "enum"
          },
          "value": null,
          "readable": true,
          "writable": true,
          "required": true
        }
      ],
      "task": null
    },
    "deptLeaderPass": {
      "true": "同意",
      "false": "不同意"
    }
  },
  "message": "读取Task的表单成功"
}
```

##  2.8、办理任务，提交task，并保存 form

这个节点代码是公用的，每个处理节点都是使用这个方法来提交本节点需要提供的表单数据，并办理任务。

```java
 /**
  * 办理任务，提交task，并保存form
  */
@PostMapping(value = "/task/complete/{taskId}")
@ResponseBody
@ApiOperation(value = "办理任务，提交task，并保存form",notes = "办理任务，提交task，并保存form")
public CommonResponse completeTask(@PathVariable("taskId") String taskId, HttpServletRequest request) {
    processService.completeTask(taskId,request);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("任务办理成功").data(taskId);
}

@Override
public void completeTask(String taskId, HttpServletRequest request) {
    Map<String, String> formProperties = new HashMap<String, String>();
    // 从request中读取参数然后转换
    Map<String, String[]> parameterMap = request.getParameterMap();
    Set<Map.Entry<String, String[]>> entrySet = parameterMap.entrySet();
    for (Map.Entry<String, String[]> entry : entrySet) {
        String key = entry.getKey();
        // fp_的意思是form paremeter
        if (StringUtils.defaultString(key).startsWith("fp_")) {
            formProperties.put(key.split("_")[1], entry.getValue()[0]);
        }
    }
    log.debug("start form parameters: {}", formProperties);
    //TODO 此处应该添加获取当前操作人的代码,先写死用 leaderuser。
    UserQueryImpl user = new UserQueryImpl();
    user = (UserQueryImpl)identityService.createUserQuery().userId("leaderuser");

    // 用户未登录不能操作，实际应用使用权限框架实现，例如Spring Security、Shiro等
    if (user == null || StringUtils.isBlank(user.getId())) {
        throw new CommonException("用户未登录不能操作");
    }
    try {
        identityService.setAuthenticatedUserId(user.getId());
        formService.submitTaskFormData(taskId, formProperties);
    } finally {
        identityService.setAuthenticatedUserId(null);
    }
}

```

我们通过这个任务：

![image-20200611101301369](/images/activiti/activiti6-08/image-20200611101301369.png)

查看当前流程图，可以看到已经到达人事节点。

![image-20200611101326298](/images/activiti/activiti6-08/image-20200611101326298.png)

## 2.9、人事签收及办理任务

这个过程和部门经理审批类似，不再详细介绍，注意每个节点表单信息是不一样的。

**步骤：获取待办列表（taskId)—签收任务—完成任务**

![image-20200611103959430](/images/activiti/activiti6-08/image-20200611103959430.png)

## 2.10 销假

销假环节，和前2步大同小异，需要注意的是，销假环节的审批用户是`kafeitu`

![image-20200611104713878](/images/activiti/activiti6-08/image-20200611104713878.png)

`applyUserId`是在流程发起环节就已经定义好的。所以，此处无需签收任务。

我们获取发起人的待办列表，可以看到：无需签收动作，assignee就已经自动指向了`kafeitu`

![image-20200611104920139](/images/activiti/activiti6-08/image-20200611104920139.png)

完成销假：

![image-20200611105355237](/images/activiti/activiti6-08/image-20200611105355237.png)

## 2.11 流程结束

![image-20200611105438350](/images/activiti/activiti6-08/image-20200611105438350.png)

# 3、总结

我们完整的走了一遍流程，对动态表单的执行过程有了全面的了解。

弊端：

1、每个流程表单信息是不一样的，我们每次获取表单信息后，利用这些信息生成我们的页面，这个其实不是很方便。表单少些还好，如果很多，那么就是灾难。和流程引擎的耦合度太高了。

2、流程业务数据全部保存在Activiti的表（act_hi_detail）中，不利于我们后期的数据处理。

