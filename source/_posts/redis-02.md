---
title: 使用docker安装redis主从模式
date: 2020-02-09 16:26:11
tags: [Docker,Redis]
categories: [Docker,Redis]
description: 使用docker安装redis 主从模式
typora-root-url: ..
---
## redis主从复制

前一节我们搭建了单体redis应用，假设我们生产环境使用了一台redis，redis挂了怎么办？
<font color=red>redis主从复制</font>：是备份关系，我们操作主库，数据会同步到从库。

那么redis主从复制有什么作用呢？
* 故障恢复：当主节点故障时，可以由从节点提供服务，快速的故障转移
* 负载均衡：可以在主从复制的基础上，配合读写分离，由主节点负责写，从节点负责读取数据，从而分担服务器的负载压力，大大提高redis的并发量。

本节我们搭建redis 一主二从，读写分离。redis主从是实现redis集群和redis哨兵高可用的基础.

### 创建工作目录
```
mkdir -p /usr/local/redis/master-slaver/master/conf
mkdir -p /usr/local/redis/master-slaver/master/data
mkdir -p /usr/local/redis/master-slaver/slaver/conf
mkdir -p /usr/local/redis/master-slaver/slaver/data
mkdir -p /usr/local/redis/master-slaver/slaver1/conf
mkdir -p /usr/local/redis/master-slaver/slaver1/data
```

### 端口约定

|角色|端口|密码|容器名称|
|:---|:---:|---:|---:|
|master |6380  |123456 |redis-6380|
|slaver |6381  |123456 |redis-6381|
|slaver |6382  |123456 |redis-6382|

### 准备redis.conf

1. 准备一份redis.conf文件 [https://redis.io/download](https://redis.io/download)。复制3份，分别存放到：
/usr/local/redis/master-slave/master/conf
/usr/local/redis/master-slave/slaver/conf
/usr/local/redis/master-slave/slaver/conf

2.创建自定义网桥
```
docker network inspect redis-master-slaver
```
这样我们容器间就可以使用容器名来通信了，避免出现ip变化后主从容器无法通信问题。

3. 分别修改这3份redis.conf的以下内容
master:
```
requirepass 123456 #给redis设置密码
bind 0.0.0.0 # 表示任意ip可连
appendonly yes #开启redis aof持久化
```
slaver:
```
requirepass 123456 #给redis设置密码
bind 0.0.0.0 # 表示任意ip可连
appendonly yes #开启redis aof持久化
slaveof redis-6380 6379 #master容器内的ip端口
masterauth 123456 #master节点的密码
```
slaver1:
```
requirepass 123456 #给redis设置密码
bind 0.0.0.0 # 表示任意ip可连
appendonly yes #开启redis aof持久化
slaveof redis-6380 6379 #master容器内的ip端口
masterauth 123456 #master节点的密码
```
注意：
* slaveof redis-6380 6379 这里指定的master端口6379是容器内的端口，不能指定为6380，否则slaver无法连接master。
* daemonize 不能设置为yes，否则无法启动。参考:[https://blog.csdn.net/Mr_Yang__/article/details/81906691](https://blog.csdn.net/Mr_Yang__/article/details/81906691)

### 创建redis容器

* master 主库
```
docker run -d -p 6380:6379 --name redis-6380 --net=redis-master-slaver \
-v /usr/local/redis/master-slaver/master/conf/redis.conf:/etc/redis/redis.conf \
-v /usr/local/redis/master-slaver/master/data:/data redis:5.0.7 \
redis-server /etc/redis/redis.conf
```
* slaver 从库
```
docker run -d -p 6381:6379 --name redis-6381 --net=redis-master-slaver \
-v /usr/local/redis/master-slaver/slaver/conf/redis.conf:/etc/redis/redis.conf \
-v /usr/local/redis/master-slaver/slaver/data:/data redis:5.0.7 \
redis-server /etc/redis/redis.conf
```
* slaver1 从库
```
docker run -d -p 6382:6379 --name redis-6382 --net=redis-master-slaver \
-v /usr/local/redis/master-slaver/slaver1/conf/redis.conf:/etc/redis/redis.conf \
-v /usr/local/redis/master-slaver/slaver1/data:/data redis:5.0.7 \
redis-server /etc/redis/redis.conf
```
### 检验功能
#### 查看master容器
![redis-6380-info.png](/images/redis/redis-6380-info.png)

可以看到role:master，以及2个slaver。我们添加一个String 类型的键值对

#### 查看slaver1容器
![redis-6381-info.png](/images/redis/redis-6381-info.png)

可以看到role:slave，可以看到master节点信息。
通过 get s1 命令可以获取到1234qwer ,说明我们同步了master主节点数据。
通过 set s2 123 时，报(error) READONLY You can't write against a read only replica.说明从节点是只读的。

#### 查看slaver1容器
![redis-6382-info.png](/images/redis/redis-6382-info.png)
slaver2和slaver1相同。

### 树结构
我们来查看一下目前的树结构,可以看到主从3个redis容器的配置文件，及产生的持久化文件。
```
[root@localhost master-slaver]# tree
.
├── master
│   ├── conf
│   │   └── redis.conf
│   └── data
│       ├── appendonly.aof
│       └── dump.rdb
├── slaver
│   ├── conf
│   │   └── redis.conf
│   └── data
│       ├── appendonly.aof
│       └── dump.rdb
└── slaver1
    ├── conf
    │   └── redis.conf
    └── data
        ├── appendonly.aof
        └── dump.rdb

9 directories, 9 files

```
### 异常情况
我们这里模拟下可能发生的异常情况。

#### master节点损坏

我们关掉master：redis-6380，模式主节点损坏情况，此时运维人员准备先将redis-6381，设置为主节点
```
docker stop redis-6380
```
修改redis-6381的redis.conf文件,注释掉下面2行
```
# slaveof redis-6380 6379 
# masterauth 123456 
```
此时我们修改redis-6382的redis.conf文件的以下内容
```
slaveof redis-6381 6379
```
重新启动redis-6381、redis-6382
```
docker restart redis-6381
docker restart redis-6382
```
#### 查看redis-6381

![master-6381-info.png](/images/redis/master-6381-info.png)

可以看到redis-6381已经升级为 master节点。

#### 查看redis-6382

![slaver-6382-info.png](/images/redis/slaver-6382-info.png)

这样我们就实现了redis主从复制，读写分离。主从模式保证了数据备份，发生故障依然需要运维人员施工。

参考：
https://blog.csdn.net/qq_28804275/article/details/80907796
https://www.cnblogs.com/demingblog/p/10295236.html