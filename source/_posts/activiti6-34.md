---
title: Activiti（34） 使用Swagger生成静态接口文档
date: 2020-10-23 09:19:39
tags: [Activiti,Swagger]
categories: [Activiti,Swagger]
description: Activiti（34） 使用Swagger生成静态接口文档
typora-root-url: ..
---

实际工作中往往需要我们提供接口文档，但是有的用户并不能直接访问Swagger-UI 来获取接口，所以本节我们介绍一下如何使用Swagger生成静态接口文档。

我们以`workflow-activit-rest`工程为例。本节我们演示生成**HTML、MarkDown、PDF**格式的静态接口文档。

## 1、添加依赖

```xml
<!--swagger静态化输出依赖-->
<dependency>
    <groupId>io.github.swagger2markup</groupId>
    <artifactId>swagger2markup</artifactId>
    <version>1.3.3</version>
</dependency>
```

另外还需要在POM文件中添加如下内容：

```xml
<repositories>
    <repository>
        <id>spring-libs-milestone</id>
        <url>https://repo.spring.io/libs-milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
    <!-- jhipster-needle-maven-repository -->
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>spring-plugins-release</id>
        <url>https://repo.spring.io/plugins-release</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>

<!--swagger静态化插件 先执行测试类生成.adoc文件再运行maven命令 asciidoctor:process-asciidoc生成html-->
<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <version>1.5.6</version>
    <configuration>
        <sourceDirectory>./docs/asciidoc/generated</sourceDirectory>
        <outputDirectory>./docs/asciidoc/html</outputDirectory>
        <headerFooter>true</headerFooter>
        <doctype>book</doctype>
        <backend>html</backend>
        <sourceHighlighter>coderay</sourceHighlighter>
        <attributes>
            <!--菜单栏在左边-->
            <toc>left</toc>
            <!--多标题排列-->
            <toclevels>3</toclevels>
            <!--自动打数字序号-->
            <sectnums>true</sectnums>
        </attributes>
    </configuration>
</plugin>
```

## 2、生成接口

此处我们写一个单元测试类来存放生成接口：

```java
package com.sxdx.workflow.activiti.rest;

import io.github.swagger2markup.GroupBy;
import io.github.swagger2markup.Language;
import io.github.swagger2markup.Swagger2MarkupConfig;
import io.github.swagger2markup.Swagger2MarkupConverter;
import io.github.swagger2markup.builder.Swagger2MarkupConfigBuilder;
import io.github.swagger2markup.markup.builder.MarkupLanguage;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.net.URL;
import java.nio.file.Paths;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
public class SwaggerTests {

    /**
     * 生成Confluence格式文档
     *
     * @throws Exception
     */
    @Test
    public void generateConfluenceDocs() throws Exception {
        //  输出Confluence使用的格式
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.CONFLUENCE_MARKUP)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();

        Swagger2MarkupConverter.from(new URL("http://localhost:9090/v2/api-docs"))
                .withConfig(config)
                .build()
                .toFolder(Paths.get("./docs/confluence/generated"));
    }

    /**
     * 生成AsciiDocs格式文档,并汇总成一个文件
     *
     * @throws Exception
     */
    @Test
    public void generateAsciiDocsToFile() throws Exception {
        //  输出Ascii到单文件
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.ASCIIDOC)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();

        Swagger2MarkupConverter.from(new URL("http://localhost:9090/v2/api-docs"))
                .withConfig(config)
                .build()
                .toFile(Paths.get("./docs/asciidoc/generated/all"));
    }


    /**
     * 生成Markdown格式文档,并汇总成一个文件
     *
     * @throws Exception
     */
    @Test
    public void generateMarkdownDocsToFile() throws Exception {
        //  输出Markdown到单文件
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.MARKDOWN)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();

        Swagger2MarkupConverter.from(new URL("http://localhost:9090/v2/api-docs"))
                .withConfig(config)
                .build()
                .toFile(Paths.get("./docs/markdown/generated/all"));
    }
}

```

其中的`http://localhost:9090/v2/api-docs`即Swagger页面的文档地址：

![image-20201023095034280](/images/activiti6-34/image-20201023095034280.png)

**说明**：此处我们需要修改SwaggerConfig文件的内容，使swagger不再扫描Activici自带的接口，否则生成的接口文档只包含Activici接口，我们自己写的接口排版是错误的。

![image-20201023095424805](/images/activiti6-34/image-20201023095424805.png)

### 2.1 生成MarkDown接口文档

首先，我么需要把服务关掉，否则会报端口占用：

![image-20201023095920423](/images/activiti6-34/image-20201023095920423.png)

然后执行单元测试方法：

![image-20201023100010900](/images/activiti6-34/image-20201023100010900.png)

执行完成后会在当前项目目录下生成如下内容：

![image-20201023100146296](/images/activiti6-34/image-20201023100146296.png)

我们使用Typora打开这个md文件：

![image-20201023100332021](/images/activiti6-34/image-20201023100332021.png)

可以看到已经生成了接口文档

### 2.2 生成PDF接口文档

这里我们是使用Typora的功能来生成PDF文件的。点击Typora左上角的 **文件-导出-PDF**，生成文件如下：

![image-20201023101100713](/images/activiti6-34/image-20201023101100713.png)

### 2.3 生成HTML接口文档

然后执行单元测试方法（记着关闭服务）：

![image-20201023101703740](/images/activiti6-34/image-20201023101703740.png)

执行结束会生成如下文件：

![image-20201023102037614](/images/activiti6-34/image-20201023102037614.png)

然后我们打开Maven：

![image-20201023102128525](/images/activiti6-34/image-20201023102128525.png)

执行结束后，就可以看到HTML文件了：

![image-20201023102234475](/images/activiti6-34/image-20201023102234475.png)

![image-20201023102303562](/images/activiti6-34/image-20201023102303562.png)

可以看到HTML格式的接口文档为我们提供了序号，更加合理一些。 