---
title: 使用Docker创建Nginx，并部署Vue项目，同时实现Nginx日志分割
date: 2020-03-08 10:28:34
tags: [Docker,Nginx]
categories: [Docker,Nginx]
description: 使用Docker创建Nginx，并部署Vue项目，同时实现Nginx日志分割
typora-root-url: ..
---
前面的文章[docker常用命令及Dockerfile笔记](https://blog.gaoyp.cn/2020/01/19/docker-03/)中，我们介绍了Docker、Dcokerfile基本知识，并且使用Docker创建了一个Nginx容器。

本节我们主要介绍，如何介绍如何使用Docker创建Nginx，并部署Vue项目，同时实现Nginx日志分割。

为了阅读方便，我们重新介绍使用Docker创建Nginx的步骤，和[docker常用命令及Dockerfile笔记](https://blog.gaoyp.cn/2020/01/19/docker-03/)相比，有一些小的修改项。


## 创建Nginx

### 拉取镜像

拉取nginx镜像到本地：
```
docker pull nginx:latest
```

说明：docker pull 镜像名称:镜像tags。如果不加 tags，默认拉取最新镜像（latest）。

### 准备工作

1. 创建挂载目录

```
mkdir -p /usr/local/nginx/{conf,html,logs,conf.d}
```
* conf : 存放外置nginx配置文件
* html : 存放Vue包
* logs : 存放nginx日志文件
* conf.d : 存放具体项目的nginx配置文件

2. 将打包好的Vue项目包（一个名为dist的文件夹）放到 /usr/local/nginx/html 目录下。

```
[root@192 html]# ls -l /usr/local/nginx/html/
总用量 4
drwxrwxr-x. 9 admin admin 250 3月   7 17:17 dist
```

3. 在/usr/local/nginx/conf下创建 nginx.conf 文件，作为外置配置文件使用。
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

4. 在/usr/local/nginx/conf下创建 log_format.conf文件，作为<font color="red">日志分割</font>配置文件使用。
```
if ($time_iso8601 ~ "^(\d{4})-(\d{2})-(\d{2})T(\d{2}):(\d{2}):(\d{2})") {
        set $year $1;
        set $month $2;
        set $day $3;
        set $hour $4;
}
access_log /var/log/nginx/${server_name}_${year}-${month}-${day}-${hour}_access.log main;
```
如上，可以实现日志的按小时分割，生产环境一般改为按天分割即可。

<font color="red">注意：</font>此处会按小时生成日志文件，Nginx容器需要有文件生成的权限，需要将nginx.conf的第一行改为

```
user root
```

5. 创建项目配置文件

在nginx.conf中我们可以看到 include /etc/nginx/conf.d/*.conf 。Nginx会扫描这个目录下所有以.conf结尾的文件。

我个人习惯在nginx.conf设置全局配置，在/etc/nginx/conf.d/下创建单独的项目配置文件。这样更清晰一些。

在/usr/local/nginx/conf.d 目录下创建vip.conf

```
[root@192 nginx]# vim conf.d/vip.conf 

server {
    listen       80;
    server_name  vip;
    include log_format.conf;    

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}

```
* include: 引入前面我们定义的日志分割配置文件 log_format.conf
* root：表示匹配成功之后进入的目录，我们创建容器后会把/usr/local/nginx/html/dist 目录下的文件拷贝到容器的/usr/share/nginx/html目录下。
* index：表示默认的首页面

5. 创建容器

```
docker run -d -p 80:80 --name nginx80 --restart=always \
 -v /usr/local/nginx/html/dist/:/usr/share/nginx/html \
 -v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
 -v /usr/local/nginx/logs/:/var/log/nginx \
 -v /usr/local/nginx/conf.d:/etc/nginx/conf.d \
 -v /usr/local/nginx/conf/log_format.conf:/etc/nginx/log_format.conf  nginx:latest 
```
如果返回一长串字符即为成功。

参数说明：
* -d 后台运行
* -p 设置端口映射 宿主机端口：容器端口
* \-\-name 设置容器别名
* \-\-restart=always 设置容器开机启动
* -v 数据卷映射 宿主机目录：容器目录。这样方便我们管理容器配置文件及日志文件

## 效果验证

### 浏览器访问Vue项目
![vue-index.jpg](/images/nginx/vue-index.jpg)

部署Vue前端项目成功

### 查看日志
```
[root@192 logs]# ll
总用量 8
-rw-r--r--. 1 root root    0 3月   9 11:08 access.log
-rw-r--r--. 1 root root  291 3月   9 11:08 error.log
-rw-r--r--. 1 root root 2110 3月   9 11:08 vip_2020-03-09-03_access.log
```
可以看到分割的日志文件，日志分割也成功。