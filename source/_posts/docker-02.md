---
title: centos7设置docker阿里云镜像加速器
date: 2019-12-26 10:02:36
tags: Docker
categories: Docker
description: centos7设置docker阿里云镜像加速器
---
{% note info %} 国内从 DockerHub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。本节记录如何配置使用阿里云镜像仓库。{% endnote %}

## 配置使用阿里云镜像仓库下载镜像

### 注册阿里云账号，并登陆
![阿里云登录.png](/images/docker/阿里云登录.png)

### 选择容器镜像服务
![镜像服务.png](/images/docker/镜像服务.png)

### 选择如下目录，并将截图命令在centos系统上执行
![设置镜像地址.png](/images/docker/设置镜像地址.png)

### 下载镜像
此时使用 docker pull 命令下载镜像就会默认使用阿里云了。非常快。下载如下Nginx，大概用了不到5秒钟。
![下载nginx.png](/images/docker/下载nginx.png)

## 推送自定义docker镜像到阿里云
我们在上面设置了阿里云的镜像库，接下来演示如何将自定义的镜像推送到自己的阿里云镜像库中，这样就可以对外提供下载了。

### 阿里云创建一个命名空间
![命名空间.png](/images/docker/命名空间.png)

### 准备一个可用来打包的容器
先docker pull 一个Tomcat 镜像，并以此启动一个Tomcat 容器 ,  通过 http://192.168.230.129:8080/ 可以看到Tomcat容器已经启动。
![tomcat容器.png](/images/docker/tomcat容器.png)

### 将上一步的 gaoyp-tomcat 容器打包成为一个新的镜像
```
docker commit -m=“提交的描述信息” -a=“作者” 容器ID 要创建的目标镜像名:[标签名]
```
此处定义的镜像为 gaoyp/tomcat ,标签为 v1.0
![创建本地镜像.png](/images/docker/创建本地镜像.png)

### 创建镜像仓库
![创建镜像仓库.png](/images/docker/创建镜像仓库.png)

点击下一步，创建镜像仓库

![自定义tomcat镜像.png](/images/docker/自定义tomcat镜像.png)

点击管理可以看到镜像信息：
![镜像信息.png](/images/docker/镜像信息.png)

### 推送镜像（参考上图第三步）
![推送自定义镜像.png](/images/docker/推送自定义镜像.png)

## 拉取自定义镜像
```
docker pull registry.cn-beijing.aliyuncs.com/gaoyipeng/gaoyp-tomcat:v1.0
```