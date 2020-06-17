---
title: docker常用命令及Dockerfile笔记
date: 2020-01-19 17:22:19
tags: [Docker,Nginx]
categories: [Docker,Nginx]
description: docker常用命令及Dockerfile笔记
typora-root-url: ..
---
> docker主要的3个概念：镜像（image）+容器（container）+仓库（repository）

*   docker镜像：概念类似虚拟机的镜像，可以用来创建新的容器。
*   docker仓库：docker仓库概念和git类似。docker仓库是用来包含镜像的位置。
*   docker容器：是由docker镜像创建的运行实例。docker容器类似虚拟机，可以执行包含启动，停止，删除等。每个容器间是相互隔离的。容器中会运行特定的运用，包含特定应用的代码及所需的依赖文件。可以把容器看作一个简易版的linux环境（包含root用户权限，进程空间，用户空间和网络空间等）和运行在其中的应用程序。

![仓库-镜像-容器关系图.png](/images/docker/仓库-镜像-容器关系图.png)

我们在这个过程中熟悉经常使用的命令。
## Docker 镜像和容器

### 镜像查询

docker镜像查询地址dockerhub：[https://hub.docker.com/search](https://hub.docker.com/search)
![hub.docker.com.png](/images/docker/hub.docker.com.png)

我们也可以使用docker search:镜像名称  的方式，搜索镜像,这种方式无法查询镜像tags。
![搜索镜像.png](/images/docker/搜索镜像.png)

各个选项说明:

* NAME: 镜像仓库源的名称
* DESCRIPTION: 镜像的描述
* OFFICIAL: 是否 docker 官方发布
* stars: 类似 Github 里面的 star，表示点赞、喜欢的意思。
* AUTOMATED: 自动构建。

### 拉取镜像
我们以nginx 为例

*   拉取nginx镜像到本地

docker pull 镜像名称:镜像tags。如果不加 tags，默认拉取最新镜像（latest）。
```
docker pull nginx:latest
```

直接执行命令报：
Got permission denied while trying to connect to the Docker daemon socket ....: connect: permission denied

我们使用su root 命令切换到root用户，再次执行docker pull nginx:latest。
![下载nginx镜像.png](/images/docker/下载nginx镜像.png)

使用docker images 命令即可查看已经拥有的镜像。
![查看已下载images.png](/images/docker/查看已下载images.png)

各个选项说明:

* REPOSITORY：表示镜像的仓库源
* TAG：镜像的标签
* IMAGE ID：镜像ID
* CREATED：镜像创建时间
* SIZE：镜像大小

### 删除镜像

如果不加 :镜像tags，默认删除最新镜像。
```
docker rmi nginx:latest
```

### 创建容器

使用镜像创建一个 nginx容器

1. 创建挂载目录
mkdir -p /usr/local/nginx/{conf,html,logs}
2. 在/usr/local/nginx/conf下创建nginx.conf文件，作为外置配置文件使用。这样我们就可以很方便的配置nginx。
```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```
在/usr/local/nginx/html目录下新建index.html
```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Docker Nginx</title>
</head>
<body>
    <h1>Docker Nginx</h1>
    <p> Hello, Nginx.</p>
</body>
</html>
```
执行命令：
```
docker run -d -p 80:80 --name nginx80 --restart=always -v /usr/local/nginx/html:/usr/share/nginx/html -v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /usr/local/nginx/logs/:/var/log/nginx nginx:latest
```

如果返回一长串字符即为成功。使用docker ps 即可看到已启动容器列表：
![docker-nginx80.png](/images/docker/docker-nginx80.png)

参数说明：
* -d 后台运行
* -p 设置端口映射 宿主机端口：容器端口
* \-\-name 设置容器别名
* \-\-restart=always 设置容器开机启动
* -v 数据卷映射 宿主机目录：容器目录。这样方便我们管理容器配置文件及日志文件

最后加上nginx:latest，表示我们是以nginx:latest为模板创建的。

浏览器访问：http://192.168.85.128/ 返回
![nginx-首页.png](/images/docker/nginx-首页.png)

### 基于容器构建镜像

镜像创建有2种方式：
* 从已经创建的容器中更新镜像，并且提交这个镜像
* 使用 Dockerfile 指令来创建一个新的镜像

我们先介绍第一种。
在上一步已经创建了一个nginx容器，使用命令docker commit 可创建镜像。
```
docker commit -m="garnett nginx" -a="garnett" nginx80 garnett/nginx:1.0
```
参数介绍：
-m 设置注释 
-a 设置作者
nginx80 容器名称，也可以使用container id
garnett/nginx:1.0  设置镜像的仓库源repository和tags

使用docker images 查看镜像列表：
![容器创建镜像.png](/images/docker/容器创建镜像.png)

### 常用命令总结

1. docker exec -it <容器ID或名字> /bin/bash   进入容器命令，有个交互式 Shell
2. docker ps -a                查看所有的容器命令
3. docker stop <容器ID或名字>   停止容器
4. docker start <容器ID或名字>  启动容器
5. docker restart <容器ID或名字> 重启容器
6. docker logs -f <容器ID或名字> 输出容器日志 -f: 让 docker logs 像使用 tail -f 一样来输出容器内部的标准输出。
7. docker inspect <容器ID或名字> 返回一个 JSON 文件记录着 Docker 容器的配置和状态信息
8. docker images 查看镜像列表
9. docker rmi <镜像ID>  删除镜像
10. docker rm <容器ID或名字>  删除容器
11. docker pull 镜像名：tags  拉取镜像
12. docker run ... 使用镜像构建容器
13. docker commit ... 使用容器构建镜像

## Dockerfile

我们在上面已经介绍过，Dockerfile可以用来构造镜像，包含了一条条构建镜像所需的指令和说明。

### 基于Dockerfile构建镜像
#### 构建nginx的镜像

我们先构建一个nginx的镜像。然后再介绍Dockerfile语法。
新建一个目录： mkdir -p /usr/local/mydockerfile/nginx，我们在这个目录下创建一个nginx的镜像。
使用vim 命令新建一个名为 Dockerfile 文件
```
FROM nginx:latest
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html
```
FROM 表示基于nginx:latest 的基础镜像。如果本地没有会使用docker pull 自动下载。
RUN 用于执行后面跟着的命令行命令。有以下俩种格式：

```
shell 格式：
RUN <命令行命令>
# <命令行命令> 等同于，在终端操作的 shell 命令。

exec 格式：
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```

注意： FROM 、RUN 等指令需要大写。

接下来我们在/usr/local/mydockerfile/nginx目录下执行

```
docker build -t nginx:1.1 .
```
docker build 命令经常用到-t -f 参数

-t 指定镜像名称及tag
-f 指定 Dockerfile文件地址，因为我们是在/usr/local/mydockerfile/nginx目录下执行的，已经包含了Dockerfile 文件，所以上面的命令不需要-f

注意：最后的 . 代表本次执行的上下文路径。上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。
上下文路径下不要放无用的文件，因为会一起打包发送给 docker 引擎，如果文件过多会造成过程缓慢。

可以看到构建过程及构建好的镜像文件。
![dockerfile构建nginx镜像.png](/images/docker/dockerfile构建nginx镜像.png)

我们使用刚才构建好的镜像创建容器：
```
docker run -d -p 82:80 --name nginx82 nginx:1.1
```
![nginx82.png](/images/docker/nginx82.png)乱码是因为写入容器的/usr/share/nginx/html/index.html 中只包含'这是一个本地构建的nginx镜像'，不是一个正常的html。至此表示我们构建的镜像是可以使用的。

### Dockerfile指令详解

1. FROM
   位于Dockerfile开头，表示基于什么镜像构建
```
FROM nginx:latest
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html
```

2.  LABEL
   Dockerfile的元数据，描述作用
```
FROM nginx:latest
LABEL version="1.0" author="garnett" description="nginx port 82"
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html
```

3.  RUN
   运行命令，每次run都会生成一个图层，所以最好使用 && 将命令合并
```
FROM nginx:latest
LABEL version="1.0" author="garnett" description="nginx port 82"
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html
RUN echo '追加。。。' > /usr/share/nginx/html/index.html
```

    可以修改为：

```
FROM nginx:latest
LABEL version="1.0" author="garnett" description="nginx port 82"
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html \
&& echo '追加。。。' > /usr/share/nginx/html/index.html
```

```
4.  ADD 和 COPY 
    ADD 和COPY都可以将上下文目录中复制文件或者目录到容器里指定路径。
    语法：
        ADD  <源路径1>  <目标路径>
        COPY <源路径1>  <目标路径>
```
ADD ADD test test/
COPY ADD test test/
```
    区别：
    ADD 添加的文件是压缩文件的话，会自动解压。
    COPY 只能复制构建目录下的文件，ADD可以添加一个构建上下文中的文件或目录，也可以是一个URL，如：

```
ADD http://wordpress.org/latest.zip /
```

5. CMD
    类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:
    *  CMD 在docker run 时运行。
    *  RUN 是在 docker build。
    *  docker run指定了其他命令，CMD命令会被忽略。
    *  定义了多个CMD，只有最后一个有效。

```
CMD echo "hello docker"
CMD echo "hello garnett"
```
构建镜像并运行docker run 
输出hello garnett

运行 docker run -it [image] /bin/bash 则没有输出。因为/bin/bash覆盖了CMD 指令。

6. ENTRYPOINT
    类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖
    * 设置容器启动时运行的命令。
    * 不会被忽略，一定会执行。
    * 一般写一个shell脚本作为ENTRYPOINT。
    * 如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。

```
FROM ubuntu
ENTRYPOINT ["/bin/ls"]
CMD []
```

7. ENV 
  环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。

```
ENV MYSQL_VERSION 5.7
RUN apt-get install -y mysql-server="${MYSQL_VERSION}"
```
8. WORKDIR
指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

```
FROM ubuntu
WORKDIR /test # 没有则自动创建test目录
WORKDIR demo
RUN pwd
```
输出 /test/demo。
9. VOLUME
定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。


* 避免重要的数据，因容器重启而丢失，这是非常致命的。
* 避免容器不断变大。
* 例如我们用nginx搭建mysql服务，数据文件可以挂到宿主机。以免因容器删除而都是数据

格式：
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>

在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。

10. EXPOSE
    仅仅只是声明端口。

    * 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
    * 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。
    格式：
    EXPOSE <端口1> [<端口2>...]



参考链接：
https://www.cnblogs.com/baizhanshi/p/9655102.html
https://www.runoob.com/docker
```