---
title: Spring Boot整合WebService
date: 2020-11-09 15:14:14
tags: [WebService,SpringBoot]
categories: [WebService,SpringBoot]
description: Spring Boot整合WebService
typora-root-url: ..
---

本节记录下Springboot2如何集成WebService.为了方便，就直接在《Spring Boot整合Dubbo》的代码基础上进行整合了。

> WebService是一种跨编程语言,跨操作系统平台的远程调用技术。
>
> 即使服务端和客户端程序的编程语言、操作系统不同,客户端上的程序也可以调用到服务端接口提供的方法。

## 1、WebService的三个规范

### 1.1 JAX-WS(Java API for XML-Based Webservices)规范

采用基于http应用层的协议Soap(Simple Object Access Protocol)，Soap协议传输的是xml数据,xml是WebService可以跨操作系统平台的基础,因为xml主要的优点就是它既与平台无关,也与厂商无关。

W3C为WebService制定了一套传输数据类型，使用xml进行描述,即XSD(XML Schema Datatypes)，任何编程语言写的WebService接口在发送数据时都要转换成WebService标准的XSD发送。

**JAX-WS三要素**

- SOAP:基于HTTP协议,采用XML格式,用来传递信息的格式
- WSDL:用来描述如何访问具体服务的文档(JAX-WS可自动生成对外接口的\说明文档,但此文档并不是给程序员提供的,而是给JDK提供的)
- UDDI:用户可以按UDDI标准搭建UDDI服务器,用来管理分发查询WebService(类似目录收集)

### 1.2 JAX-RS规范

是JAVA针对**REST风格**制定的一套Web服务规范,rest风格的WebService不采用SOAP协议传输,而是采用http协议,可以返回xml或json.

### 1.3 JAXM&SAAJ

编码麻烦,暴露SOAP的底层细节,废弃。

****



Apache CXF(Celtix+Xfire)是一个开源的WebService框架，支持多种协议，包括SOAP、XML/HTTP、**RESTFUL**，正是因为CXF支持RESTFUL风格,所以CXF也是支持JAX-RS服务规范的框架.

![image-20201110144604411](/images/webservice-01/image-20201110144604411.png)

## 2、WebService注解大全

后续代码中的注解不明白的可以参考这个：

```
WebService的注解都位于javax.jws包下: 

@WebService-定义服务，在类上边
    targetNamespace：指定命名空间
    name：portType的名称
    portName：port的名称 wsdl:portName。缺省值为 WebService.name+Port。
    serviceName：服务名称
    endpointInterface：SEI接口地址，如果一个服务类实现了多个接口，只需要发布一个接口的方法，可通过此注解指定要发布服务的接口。 

@WebMethod-定义方法，在公开方法上边
    operationName：方法名
    exclude：设置为true表示此方法不是webservice方法，反之则表示webservice方法，默认是false 

@WebResult-定义返回值，在方法返回值前边
    name：返回结果值的名称 

@WebParam-定义参数，在方法参数前边
    name：指定参数的名称
```



## 3、添加依赖

1、首先在父工程中定义我们需要的Jar包：

```xml
<fastjson.version>1.2.70</fastjson.version>
<webservice.cxf.version>3.2.4</webservice.cxf.version> 

<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web-services</artifactId>
     <version>${spring-boot.version}</version>
</dependency>
<!--这个包是给webservice发布使用的。-->
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
    <version>${webservice.cxf.version}</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>${fastjson.version}</version>
</dependency>
```

2、在server-provider中导入Jar包

```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web-services</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
</dependency>
```

3、在service-consumer中添加Jar包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web-services</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
</dependency>
```



## 4、发布服务

首先我们定义一个发布接口：

```java
package com.sxdx.service.provider.service;

import javax.jws.WebMethod;
import javax.jws.WebParam;
import javax.jws.WebService;

@WebService(targetNamespace = "http://WebServiceDemoServiceImpl.impl.service.provider.service.sxdx.com")
public interface WebServiceDemoService {

    @WebMethod
    public String helloMethod(@WebParam(name = "name", targetNamespace = "http://WebServiceDemoServiceImpl.impl.service.provider.service.sxdx.com") String name);
}

```

@WebService：标明是个webservice服务，发布的时候会带上这个类。
@WeBMethod：webservice中发布的方法
@WebParam：对参数的别名，不写也不影响，但是参数在wsdl中看起来是arg0，不利于理解。

它的实现类：

```java
package com.sxdx.service.provider.service.impl;

import com.sxdx.service.provider.service.WebServiceDemoService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.jws.WebMethod;
import javax.jws.WebParam;
import javax.jws.WebService;

/**
 * @program: dubbo-boot
 * @description:
 * @author: garnett
 * @create: 2020-11-10 10:41
 **/
@Slf4j
@Component
@WebService(name = "webServiceDemoService",
        targetNamespace = "http://WebServiceDemoServiceImpl.impl.service.provider.service.sxdx.com",
        endpointInterface = "com.sxdx.service.provider.service.WebServiceDemoService")
public class WebServiceDemoServiceImpl implements WebServiceDemoService {
    @Override
    public String helloMethod(String name) {
        return "你好啊："+name;
    }
}

```

然后发布(此处使用CXF发布服务)：

```java
package com.sxdx.service.provider.config;


import com.sxdx.service.provider.service.WebServiceDemoService;
import org.apache.cxf.Bus;
import org.apache.cxf.bus.spring.SpringBus;
import org.apache.cxf.jaxws.EndpointImpl;
import org.apache.cxf.transport.servlet.CXFServlet;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.xml.ws.Endpoint;

@Configuration
public class WebServiceConfig {
    @Autowired
    private WebServiceDemoService webServiceDemoService;


    //* 此方法被注释后:wsdl访问地址为http://127.0.0.1:8080/services/user?wsdl
    //* 去掉注释后：wsdl访问地址为：http://127.0.0.1:8080/webservice/user?wsdl
    @Bean(name = "cxfServlet")
    public ServletRegistrationBean cxfServlet() {
        return new ServletRegistrationBean(new CXFServlet(), "/webservice/*");
    }

    @Autowired
    @Qualifier(Bus.DEFAULT_BUS_ID)
    private SpringBus bus;

    @Bean
    public Endpoint webServiceDemoEndPoint() {
        EndpointImpl endpoint = new EndpointImpl(bus, webServiceDemoService);
        endpoint.publish("/webServiceDemoService");
        return endpoint;
    }
}
```

启动服务访问:http://localhost:1111/webservice/webServiceDemoService?wsdl

![image-20201110135925051](/images/webservice-01/image-20201110135925051.png)

证明发布成功了。

### 3.1 **WSDL**文件解析

**WSDL文档应该从下往上阅读。**

1.先看service标签，看相应port的binding属性，然后通过值查找上面的binding标签。

> Service：相关端口的集合，包括其关联的接口、操作、消息等。

2.通过binding标签可以获得具体协议等信息，然后查看binding的type属性

> Binding：特定端口类型的具体协议和数据格式规范

3.通过binding的type属性，查找对应的portType，可以获得可操作的方法和参数、返回值等。

> portType: 服务端点，描述 web service可被执行的操作方法，以及相关的消息，通过binding指向portType

4.通过portType下的operation标签的message属性，可以向上查找message获取具体的数据参数信息。

> message: 定义一个操作（方法）的数据参数

5.types: 定义 web service 使用的全部数据类型



## 4、消费服务

webservice接口的调用方式有很多（axis、axis2、cxf、wsimport生成客户端方式等），这里介绍2种（axis、cxf）：

### 4.1 Axis 调用方式

首先在service-consumer中添加axis的Jar包

```xml

<dependency>
    <groupId>org.apache.axis</groupId>
    <artifactId>axis</artifactId>
    <version>1.4</version>
</dependency>
<dependency>
    <groupId>org.apache.axis</groupId>
    <artifactId>axis-jaxrpc</artifactId>
    <version>1.4</version>
</dependency>
<dependency>
    <groupId>commons-discovery</groupId>
    <artifactId>commons-discovery</artifactId>
    <version>0.2</version>
</dependency>
```

添加测试类

```java
package com.sxdx.service.consumer;

import com.alibaba.fastjson.JSONObject;
import org.apache.axis.client.Call;
import org.apache.axis.client.Service;

import javax.xml.namespace.QName;

/**
 * @program: dubbo-boot
 * @description:
 * @author: garnett
 * @create: 2020-11-10 12:00
 **/

public class Test {

    private static final String SERVICEURL = "http://localhost:1111/webservice/webServiceDemoService?wsdl";
    private static final String TARGETNAMESPACE = "http://WebServiceDemoServiceImpl.impl.service.provider.service.sxdx.com";
    private static String name = "Garnett";

    /**
     * axis方式调用 Webservice 服务
     */
    @org.junit.Test
    public void test1(){
        try {
            Service service = new Service();

            Call call = (Call) service.createCall();
            call.setTargetEndpointAddress(SERVICEURL);
            call.setOperationName(new QName(TARGETNAMESPACE,"helloMethod"));//WSDL里面描述的接口名称
            //接口的参数说明
            call.addParameter(new QName(TARGETNAMESPACE,"name"),
                    org.apache.axis.encoding.XMLType.XSD_STRING,
                    javax.xml.rpc.ParameterMode.IN);
            ////给方法传递参数，并且调用方法
            String result = (String)call.invoke(new Object[]{name});
            System.out.println("result is " + result);

        } catch (Exception e) {
            System.err.println(e.toString());
        }
    }
}

```

调用结果如下：

![image-20201110161115767](/images/webservice-01/image-20201110161115767.png)

4.2 CXF 调用方式

```java
package com.sxdx.service.consumer;

import com.alibaba.fastjson.JSONObject;
import org.apache.axis.client.Call;
import org.apache.axis.client.Service;
import org.apache.cxf.endpoint.Client;
import org.apache.cxf.jaxws.endpoint.dynamic.JaxWsDynamicClientFactory;

import javax.xml.namespace.QName;

/**
 * @program: dubbo-boot
 * @description:
 * @author: garnett
 * @create: 2020-11-10 12:00
 **/

public class Test {

    private static final String SERVICEURL = "http://localhost:1111/webservice/webServiceDemoService?wsdl";
    private static final String TARGETNAMESPACE = "http://WebServiceDemoServiceImpl.impl.service.provider.service.sxdx.com";
    private static String name = "Garnett";

    /**
     * axis方式调用 Webservice 服务
     */
    @org.junit.Test
    public void test1(){
        try {
            Service service = new Service();

            Call call = (Call) service.createCall();
            call.setTargetEndpointAddress(SERVICEURL);
            call.setOperationName(new QName(TARGETNAMESPACE,"helloMethod"));//WSDL里面描述的接口名称
            //接口的参数说明
            call.addParameter(new QName(TARGETNAMESPACE,"name"),
                    org.apache.axis.encoding.XMLType.XSD_STRING,
                    javax.xml.rpc.ParameterMode.IN);
            ////给方法传递参数，并且调用方法
            String result = (String)call.invoke(new Object[]{name});
            System.out.println("result is " + result);

        } catch (Exception e) {
            System.err.println(e.toString());
        }
    }

    /**
     * CXF调用方式
     */
    @org.junit.Test
    public void test2(){
        JaxWsDynamicClientFactory dcf = JaxWsDynamicClientFactory.newInstance();
        Client client = dcf.createClient(SERVICEURL);
        String params = "{\"name\":\""+name+"\"}";
        try {
            Object[] generalAssetResults = client.invoke("helloMethod", params);
            System.out.println("AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA");
            System.out.println(generalAssetResults[0]);
        }catch (Exception e){
            System.err.println(e.toString());
        }

    }
}

```

调用结果如下：

![image-20201110162734016](/images/webservice-01/image-20201110162734016.png)



就先介绍到这里，感觉这2种就差不多够用了。其他方式需要再说吧。



参考：https://blog.csdn.net/bicheng4769/article/details/104362433