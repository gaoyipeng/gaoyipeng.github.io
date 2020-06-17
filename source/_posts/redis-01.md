---
title: 使用docker安装单点redis
date: 2020-02-08 16:58:42
tags: [Docker,Redis]
categories: [Docker,Redis]
description: 使用docker安装单点redis
typora-root-url: ..
---
## 安装单个redis

1. 拉取镜像,我使用的是5.0.7版本。[https://hub.docker.com/_/redis/?tab=tags](https://hub.docker.com/_/redis/?tab=tags)
```
docker pull redis:5.0.7
```
2. 准备一份redis.conf文件 [https://redis.io/download](https://redis.io/download)，使用 xftp 上传到 /usr/local/redis/conf下。

注意：默认docker运行的redis是不存在配置文件的，即无配置启动.
另外还可以通过wget下载redis.conf：
```
wget http://download.redis.io/redis-stable/redis.conf
```

```
mkdir /usr/local/redis/data
mkdir /usr/local/redis/conf
```
上传时发现一直报错，因为我是使用admin登录的，传输到虚拟机（服务器）的文件夹（目的地的文件夹）的权限不够。
```
chmod 777 /usr/local/redis/conf
```
再次上传,然后把权限改回来，777权限很危险，尽量不要这么设置。

修改redis.conf中的以下内容
* requirepass 123456 #给redis设置密码
* bind 127.0.0.1 # 注释掉这部分，表示任意ip可连
* appendonly yes #开启redis aof持久化

3. 创建redis容器

```
docker run -d -p 6379:6379 --name redis-6379 -v /usr/local/redis/conf/redis.conf:/etc/redis/redis.conf -v /usr/local/redis/data:/data redis:5.0.7 redis-server /etc/redis/redis.conf
```
![docker-redis-single](/images/redis/docker-redis-single.png)
* -p 6379:6379:把容器内的6379端口映射到宿主机6379端口
* -v /usr/local/redis/conf/redis.conf:/etc/redis/redis.conf:把宿主机配置好的redis.conf放到容器内的这个位置中
* -v /usr/local/redis/data:/data:把redis持久化的数据在宿主机内显示，做数据备份
* redis-server /etc/redis/redis.conf：这个是关键配置，让redis不是无配置启动，而是按照这个redis.conf的配置启动
* –appendonly yes：redis启动后数据持久化

使用redis-Desktop连接
![redis-Desktop-single.png](/images/redis/redis-Desktop-single.png)

我们使用命令向redis中插入几个String类型的值，再次查看/usr/local/redis/data目录,发现已经生成了持久化文件。

![single-data.png](/images/redis/single-data.png)

单体redis较为简单，完工！


