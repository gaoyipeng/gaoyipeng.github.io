---
title: linux（centos7）安装docker
date: 2019-12-26 10:02:36
tags: Docker
categories: Docker
description: linux（centos7）安装docker
typora-root-url: ..
---
{% note info %} Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中,然后发布到任何流行的Linux机器或Windows 机器上,也可以实现虚拟化,容器是完全使用沙箱机制,相互之间不会有任何接口。{% endnote %}

## 检查内核版本
> centos7 必须是3.10及以上: uname ‐r

![版本检查.png](/images/docker/版本检查.png)

## 安装docker前的准备
> 官网地址：https://docs.docker.com/install/linux/docker-ce/centos/

### 安装所需的软件包
```
yum install -y yum-utils \
>   device-mapper-persistent-data \
>   lvm2
```

![安装所需的软件包.png](/images/docker/安装所需的软件包.png)

### 设置存储库
```
yum-config-manager \
>     --add-repo \
>     https://download.docker.com/linux/centos/docker-ce.repo
```
![设置存储库.png](/images/docker/设置存储库.png)

## 安装docker

### 安装最新版本docker
```
yum install docker-ce
```
![安装docker.png](/images/docker/安装docker.png)

### 安装指定版本的docker

```
 yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```
其中<VERSION_STRING>可以通过以下命令查看

```
yum list docker-ce --showduplicates | sort -r
```
![查询docker版本.png](/images/docker/查询docker版本.png)

最终命令如下：
> 我们以第一个截图第一个为例，以下命令未实践，根据官网说明自己写的，所以没有截图，仅供参考，本人安装的是最新docker
```
yum install docker-ce-19.03.4.ce docker-ce-cli-19.03.4.ce containerd.io
```

### 安装成功确认
```
docker version
```
![版本查询.png](/images/docker/版本查询.png)

### docker启动、关闭

启动docker
```
systemctl start docker
```
设置开机启动
```
systemctl enable docker
```
![docker开机启动.png](/images/docker/docker开机启动.png)

关闭docker
```
systemctl stop docker
```
## 卸载
###  卸载旧版本docker
```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
### 卸载新版本
```
yum remove docker-ce
```
删除所有图像，容器和卷
```
rm -rf /var/lib/docker
```

## docker 镜像查看地址：https://hub.docker.com

![镜像查询.png](/images/docker/镜像查询.png)

## 安装docker-compose

> Docker Compose 是一个用来定义和运行复杂应用的 Docker 工具。
 使用 Docker Compose 不再需要使用 shell 脚本来启动容器。(通过 docker-compose.yml 配置)

官网地址：https://docs.docker.com/compose/compose-file/

### 安装命令

官网的安装命令太慢，有时无法连接，查阅其他人博客后发现如下安装方式：

1.  通过GitHub安装（可以通过修改 URL 中的版本，自定义需要的版本）

```
sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

2.  通过Daocloud镜像安装（可以通过修改 URL 中的版本，自定义需要的版本）

```
curl -L https://get.daocloud.io/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### 卸载docker-compose
```
sudo rm /usr/local/bin/docker-compose
```

### docker-compose常用命令
```
docker-compose up -d
```

后台启动 docker-compose 中的容器，如果没有容器会根据docker-compose内容创建容器，并启动。
```
docker-compose stop
```
停止 docker-compose 中的容器，但不会删除容器
```
docker-compose down
```
停止 docker-compose 中的容器并删除容器

参考博客：https://www.cnblogs.com/morang/p/9501223.html