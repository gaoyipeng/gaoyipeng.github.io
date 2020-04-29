---
title: docker redis集群安装
tags:
  - docker
  - redis
categories:
  - docker
  - redis
description: docker redis集群安装
date: 2020-02-12 16:28:07
---

## redis集群安装

目前为止我们的程序一直都是向1台redis写数据，其他的redis只是备份而已。实际场景中，单个redis节点可能不满足要求，因为：
* 单个redis并发有限
* 单个redis接收所有的数据，最终回导致内存太大，内存太大会导致rdb文件过大，从很大的rdb文件中同步恢复数据会很慢。

所以，我们需要redis cluster 即redis集群。

Redis 集群是一个提供在多个 Redis 节点间共享数据的程序集。提高Redis的可用性，不会因为某个节点宕掉或者不可达而导致整个集群网络的不可用。

Redis 集群的优势:
* 自动分割数据到不同的节点上。
* 整个集群的部分节点失败或者不可达的情况下能够继续处理命令。

因为Redis集群中至少应该有奇数个主节点，所以本文将创建6个Redis节点，其中3个为主节点，3个为从属节点，用于从主节点拉取数据进行备份。

### 节点规划
![docker-redis-cluster1.png](/images/redis/docker-redis-cluster1.png)

### 创建工作目录
```
mkdir -p /usr/local/redis/cluster/config
mkdir -p /usr/local/redis/cluster/data
```
* cluster 作为集群主目录
* config 存放redis配置文件
* data 存放持久化文件

### 准备redis.conf文件

下载redis.conf：
```
wget http://download.redis.io/redis-stable/redis.conf
```

复制6份redis.conf，保存到/usr/local/redis/cluster/config目录下，分别命名为node-6391.conf、node-6392.conf、node-6393.conf、node-6394.conf、node-6395.conf、node-6396.conf

![redis-confs.png](/images/redis/redis-confs.png)

修改redis-6391.conf以下内容，其他5个配置文件类似：

```
port 6391  #修改端口
bind 0.0.0.0 #设置所有ip均可访问
cluster-enabled yes #开启集群功能
cluster-config-file nodes-6391.conf #集群内部配置文件
cluster-node-timeout 15000 #节点超时时间，单位毫秒
```

### 编写 docker-compose.yml

在/usr/local/redis/cluster/下创建docker-compose.yml文件：

```
version: "3.6"
services:
  redis-master1:
    image: redis:5.0.7 # 基础镜像
    container_name: redis-master1 # 容器服务名
    working_dir: /config # 工作目录
    environment: # 环境变量
      - PORT=6391 # 跟 config/nodes-6391.conf 里的配置一样的端口
    ports: # 映射端口，对外提供服务
      - "6391:6391" # redis 的服务端口
      - "16391:16391" # redis 集群监控端口
    stdin_open: true # 标准输入打开
    networks: # docker 网络设置
      redis-master:
        ipv4_address: 172.50.0.2
    tty: true
    privileged: true # 拥有容器内命令执行的权限
    volumes:
      - /usr/local/redis/cluster/compose/config:/config
      - /usr/local/redis/cluster/data/6391:/data
    entrypoint: # 设置服务默认的启动程序
      - /bin/bash
      - redis.sh
  redis-master2:
    image: redis:5.0.7
    working_dir: /config
    container_name: redis-master2
    environment:
      - PORT=6392
    networks:
      redis-master:
        ipv4_address: 172.50.0.3
    ports:
      - "6392:6392"
      - "16392:16392"
    stdin_open: true
    tty: true
    privileged: true
    volumes:
      - /usr/local/redis/cluster/compose/config:/config
      - /usr/local/redis/cluster/data/6392:/data
    entrypoint:
      - /bin/bash
      - redis.sh
  redis-master3:
    image: redis:5.0.7
    container_name: redis-master3
    working_dir: /config
    environment:
      - PORT=6393
    networks:
      redis-master:
        ipv4_address: 172.50.0.4
    ports:
      - "6393:6393"
      - "16393:16393"
    stdin_open: true
    tty: true
    privileged: true
    volumes:
      - /usr/local/redis/cluster/compose/config:/config
      - /usr/local/redis/cluster/data/6393:/data
    entrypoint:
      - /bin/bash
      - redis.sh
  redis-slave1:
    image: redis:5.0.7
    container_name: redis-slave1
    working_dir: /config
    environment:
      - PORT=6394
    networks:
      redis-slave:
        ipv4_address: 172.30.0.2
    ports:
      - "6394:6394"
      - "16394:16394"
    stdin_open: true
    tty: true
    privileged: true
    volumes:
      - /usr/local/redis/cluster/compose/config:/config
      - /usr/local/redis/cluster/data/6394:/data
    entrypoint:
      - /bin/bash
      - redis.sh
  redis-salve2:
    image: redis:5.0.7
    working_dir: /config
    container_name: redis-salve2
    environment:
      - PORT=6395
    ports:
      - "6395:6395"
      - "16395:16395"
    stdin_open: true
    networks:
      redis-slave:
        ipv4_address: 172.30.0.3
    tty: true
    privileged: true
    volumes:
      - /usr/local/redis/cluster/compose/config:/config
      - /usr/local/redis/cluster/data/6395:/data
    entrypoint:
      - /bin/bash
      - redis.sh
  redis-salve3:
    image: redis:5.0.7
    container_name: redis-slave3
    working_dir: /config
    environment:
      - PORT=6396
    ports:
      - "6396:6396"
      - "16396:16396"
    stdin_open: true
    networks:
      redis-slave:
        ipv4_address: 172.30.0.4
    tty: true
    privileged: true
    volumes:
      - /usr/local/redis/cluster/compose/config:/config
      - /usr/local/redis/cluster/data/6396:/data
    entrypoint:
      - /bin/bash
      - redis.sh
networks:
  redis-master:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.50.0.0/16
  redis-slave:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.30.0.0/16
```

### 编写 redis 默认的启动脚本

在/usr/local/redis/cluster/config下创建 redis.sh，对应docker-compose.yml中redis.sh：
```
redis-server  /config/nodes-${PORT}.conf
```

### 最终目录结构


### 启动

```
docker-compose up -d
```
![cluster-up.png](/images/redis/cluster-up.png)

初始化集群，创建 3 主 3 从的 redis 集群
```
redis-cli --cluster create 192.168.85.128:6391 192.168.85.128:6392 192.168.85.128:6393 192.168.85.128:6394 192.168.85.128:6395 192.168.85.128:6396 --cluster-replicas 1
```