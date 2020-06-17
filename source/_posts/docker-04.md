---
title: Docker Compose笔记
date: 2020-01-26 22:12:54
tags: [Docker,ELK]
categories: [Docker,ELK]
description: Docker Compose笔记
typora-root-url: ..
---
Docker Compose 是用于定义和运行多容器 Docker 应用程序的工具。微服务架构的应用系统一般包含若干个微服务，每个微服务可能会部署可能需要用到多个Docker容器，比如MySQL，Redis，Nginx等，单独的去管理每个容器可能会比较麻烦。

Docker Compose 可以通过一个yml文件来统一管理这些容器，可以极大简化我们的应用部署过程。

## 安装Docker Compose

可以参考我的博客：[linux（centos7）安装docker](http://blog.gaoyp.cn/2019/12/26/docker-01/)

## Docker Compose 配置指令

我们先来看一个docker-compose.yml文件，这个是直接抄鸟哥的，感谢鸟哥。鸟哥博客：[https://mrbird.cc/](https://mrbird.cc/)
```
version: '3'
services:
  elasticsearch:
    image: elasticsearch:6.4.1
    container_name: elasticsearch
    environment:
      - "cluster.name=elasticsearch" #集群名称为elasticsearch
      - "discovery.type=single-node" #单节点启动
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" #jvm内存分配为512MB
    volumes:
      - /kiki/elasticsearch/plugins:/usr/share/elasticsearch/plugins
      - /kiki/elasticsearch/data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - my-bridge  
kibana:
    image: kibana:6.4.1
    container_name: kibana
    links:
      - elasticsearch:es #配置elasticsearch域名为es
    depends_on:
      - elasticsearch
    environment:
      - "elasticsearch.hosts=http://es:9200" #因为上面配置了域名，所以这里可以简写为http://es:9200
    ports:
      - 5601:5601
    networks:
      - my-bridge
  logstash:
    image: logstash:6.4.1
    container_name: logstash
    volumes:
      - /kiki/logstash/logstash-febs.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch
    links:
      - elasticsearch:es
    ports:
      - 4560:4560
    networks:
      - my-bridge

networks:
  my-bridge:
    driver: bridge
```
以上是一个ELK日志分析系统的docker-compose.yml文件。我们通过这个为例介绍常用的 Docker Compose 配置指令

1. version：指定本 yml 依从的 compose 哪个版本制定的
    compose与docker版本对应可以查看[https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)
    使用 docker version 可以查看docker版本，我虚拟机docker版本是 19.03.5,目前为止 version 3.7 及以下均可。
    ![dockercompose对应关系.png](/images/docker/dockercompose对应关系.png)
    
2. services：多个容器集合。一个service代表一个container，可以通过image创建，也可以通过本地的dockerfile来创建。上栗就创建了3个container。
3. image：使用指定容器运行的镜像
4. container_name：指定自定义容器名称，而不是生成的默认名称。
5. volumes：将主机的数据卷或着文件挂载到容器里。
    格式：  
    \- 主机主机的数据卷或着文件：容器数据卷或着文件
6. depends_on：设置依赖关系：使用docker-compose up 启动时elasticsearch启动后 kibana和logstash才会启动。关闭时反过来
7. environment：添加环境变量。您可以使用数组或字典、任何布尔值，布尔值需要用引号引起来
8. ports：进行端口映射
9. networks： 配置容器连接的网络,此处设置可名为my-bridge的bridge类型的网络。关于docker 网络，我们下一节再介绍。
10. links： 服务之间可以使用服务名称相互访问，links 允许定义一个别名，从而使用该别名访问其它服务.上面我们设置elasticsearch别名为es。 kibana和logstash可以使用elasticsearch或es访问elasticsearch容器。
11. command: 覆盖容器启动的默认命令
12. build：与image效果相同。不过build是利用Dockerfile构建，Compose 会利用它自动构建镜像，该值可以是一个路径，也可以是一个对象，用于指定 Dockerfile 参数
```
version: "3.7"
services:
  webapp:
    build: ./dir
```
指定上下文路径为 ./dir ，及使用./dir下的Dockerfile文件构建镜像。
```
version: "3.7"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
      labels:
        - "com.example.description=Accounting webapp"
        - "com.example.department=Finance"
        - "com.example.label-with-empty-value"
      target: prod
```
指定上下文路径为 ./dir ，及使用./dir下的Dockerfile文件构建镜像。
    * context：上下文路径。
    * dockerfile：指定构建镜像的 Dockerfile 文件命。这里指定 Dockerfile-alternate，不是必须命名为 Dockerfile
    * args：添加构建参数，这是只能在构建过程中访问的环境变量。
    * labels：设置构建镜像的标签。
    * target：多层构建，可以指定构建哪一层。

## DockerCompose常用命令

> docker-compose up -d

通过docker-compose.yaml文件创建并启动容器，-d，后台启动
> docker-compose stop

关闭docker-compose.yaml文件创启动的容器

> docker-compose start

启动docker-compose stop命令关闭的容器

>docker-compose down

停止并删除创建的容器（同时删除创建的network，volume，container）

> docker-compose up

当服务的配置发生更改时，可使用 docker-compose up 命令更新配置.此时，Compose 会删除旧容器并创建新容器，新容器会以不同的 IP 地址加入网络，名称保持不变，任何指向旧容起的连接都会被关闭，重新找到新容器并连接上去


## 搭建 ELK日志分析系统

接下来我们实践一下，使用DockerCompose 搭建一个ELK日志分析系统。

ELK:（ELK 由 ElasticSearch 、 Logstash 和 Kiabana 三个开源工具组成）,Elasticsearch用于存储日志信息，Logstash用于收集日志，Kibana用于图形化展示。

#### 准备工作

1. linux Elasticsearch 默认使用 mmapfs目录来存储其索引。操作系统对mmap计数的限制可能太低，这可能会导致内存不足异常。在Linux上，可以通过运行以下命令来增加限制 
```
sysctl -w vm.max_map_count=262144
```
2. 创建数据挂载路径
```
mkdir -p /usr/local/kiki/elk # 创建ELK Docker Compose文件存储路径
mkdir -p /usr/local/kiki/elasticsearch/data # 创建Elasticsearch数据挂载路径
mkdir -p /usr/local/kiki/elasticsearch/plugins # 创建Elasticsearch插件挂载路径
mkdir -p /usr/local/kiki/logstash # 创建Logstash配置文件存储路径
```
3. 在/usr/local/kiki/logstash 路径下创建logstash-kiki.conf配置文件：
```
input {
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => 4560
    codec => json_lines
  }
}
output {
  elasticsearch {
    hosts => "es:9200"
    index => "kiki-logstash-%{+YYYY.MM.dd}"
  }
}
```
logstash-kiki.conf定义了logstash的input和output。

4. 在/usr/local/kiki/elk下创建docker-compose.yml文件，内容如下，

```
version: '3'
services:
  elasticsearch:
    image: elasticsearch:6.4.1
    container_name: elasticsearch
    environment:
      - "cluster.name=elasticsearch" #集群名称为elasticsearch
      - "discovery.type=single-node" #单节点启动
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" #jvm内存分配为512MB
    volumes:
      - /usr/local/kiki/elasticsearch/plugins:/usr/share/elasticsearch/plugins
      - /usr/local/kiki/elasticsearch/data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - my-bridge  
  kibana:
    image: kibana:6.4.1
    container_name: kibana
    links:
      - elasticsearch:es #配置elasticsearch域名为es
    depends_on:
      - elasticsearch
    environment:
      - "elasticsearch.hosts=http://es:9200" #因为上面配置了域名，所以这里可以简写为http://es:9200
    ports:
      - 5601:5601
    networks:
      - my-bridge
  logstash:
    image: logstash:6.4.1
    container_name: logstash
    volumes:
      - /usr/local/kiki/logstash/logstash-kiki.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch
    links:
      - elasticsearch:es
    ports:
      - 4560:4560
    networks:
      - my-bridge
networks:
  my-bridge:
    driver: bridge
```

实际操作如图：
![elk准备工作.png](/images/docker/elk准备工作.png)

#### 启动ELK

切换到/usr/local/kiki/elk 目录下，使用docker-compose up 启动。第一次执行需要下载3个镜像。
![compose启动elk.png](/images/docker/compose启动elk.png)

使用docker-compose ps查看启动状态
![compose-ps-elk.png](/images/docker/compose-ps-elk.png)

发现elasticsearch没有启动，这是因为elasticsearch的data目录权限不够。执行 
```
chmod 774 /usr/local/kiki/elasticsearch/data/
```
再次执行docker-compose up -d,再次查看 docker-compose ps
![elk-ps.png](/images/docker/elk-ps.png)

logstash-kiki.conf配置文件中用到了json_lines来处理json类型的日志数据。所以我们需要为logstash安装json_lines插件。

* 进入到logstash 容器中：docker exec -it logstash /bin/bash
* 进入容器的 /bin/ 目录，执行logstash-plugin install logstash-codec-json_lines

![安装json_lines.png](/images/docker/安装json_lines.png)

安装过程很慢，中间报错是因为断网了，再次执行下命令静静等待就行，安装完成后输入exit退出，或者按Ctrl+Q+P 退出。

访问http://192.168.85.128:5601/ ，192.168.85.128是我虚拟机IP，结果如下：
![浏览器访问elk.png](/images/docker/浏览器访问elk.png)

### 使用ELK
我们顺便记录下SpringBoot项目如何接入ELK日志分析系统。

1. 首先在POM.xml文件中引入logstash,此处springboot项目使用logback作为日志框架
```
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>6.1</version>
</dependency>
```
2. 在项目的logback-spring.xml配资文件中添加
```
<!--输出到 logstash的 appender-->
<appender name="logstash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <destination>192.168.85.128:4560</destination>
    <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>

......
<root level="info">
    ......
    <appender-ref ref="logstash" />
</root>
```
192.168.85.128:4560对应我们刚刚搭建的Logstash地址

3. 配置Kiabana
访问http://192.168.85.128:5601/做一些配置

* Kibana管理界面点击左侧Management,点击 Kinaba Index Patterns
* 在Index pattern里输入我们在logstash配置文件logstash-kiki.conf里output.index指定的值kiki-logstash-*,点击下一步，注意，这里需要检查elasticsearch中是否有匹配数据。
所以，需要按上面的步骤创建springboot项目并启动，否则无法点击Next Step。
* 点击Next Step，在下拉框里选择@timestamp
* 点击 Create index patterns

![logstash](/images/docker/logstash.gif) 

4. 使用postman调用接口，产生日志数据,代码如下
```
package com.sxdx.sso.resource.one.controller;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.security.Principal;
import java.util.HashMap;
import java.util.Map;

@Slf4j
@RestController
public class OneController {
    @GetMapping("/user")
    public Principal user(Principal principal) {
        log.info("获取当前登录人信息");
        return principal;
    }
}
```
我的访问接口访问路径为localhost:8002/one/user。查看是否搜集到了日志数据。

![验证是否搜集到了日志数据.png](/images/docker/验证是否搜集到了日志数据.png) 

可以看到已经获取到了日志数据。完工！！！
