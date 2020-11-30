---
title: docker 数据存储笔记
date: 2020-02-04 10:22:09
tags: [Docker,Mysql]
categories: Docker
description: docker 数据存储笔记
typora-root-url: ..
---
Docker容器在运行的时候会产生数据，为了不让这些数据随着容器的删除而删除，Docker支持数据持久化。

Docker数据持久化主要有三种种方式：volume, bind mounts, tmpfs mounts

![types-of-mounts-volume.png](/images/docker/types-of-mounts-volume.png)

这张图很好的说明了三者的关系:
* volume 使用volume数据将持久化在Docker管理的volume中（保存在宿主机 /var/lib/docker/volumes目录下）
* bind mount是直接挂载的宿主机的文件目录，将宿主机的目录映射到容器中；
* tmpfs mount是将数据写入缓存，容器停止后就会被清理。这种模式不太常用，不做太多介绍。

使用bind mount，数据将持久化在我们指定的宿主机的某个目录中。挂载宿主机目录时数据覆盖有两个规则：
* 如果宿主机目录下内容为空，容器映射目录有数据，则会把容器数据复制到宿主机目录
* 如果宿主机目录下内容不为空，则会把宿主机目录下内容映射到容器里，并且容器原数据会隐藏。

## volume方式

Linux环境下，docker 默认在主机上会有一个特定的区域（/var/lib/docker/volumes/ ），该区域用来存放 volume。

    volume 可以通过 docker volume 进行管理，如创建、删除等操作。
    volume 在生成的时候如果不指定名称，便会随机生成。

我们使用docker创建一个mysql容器来验证一下，首先我们看一下mysql:5.7.29的 [Dockerfile](https://hub.docker.com/layers/mysql/library/mysql/5.7.29/images/sha256-5e443fc090c75413ffc20665ed1880c7961bc6af8ae8997510fdd7e69d8557ad)内容
```
......
VOLUME [/var/lib/mysql]
......
EXPOSE 3306 33060
CMD ["mysqld"]
```

其中的 <font color=red>VOLUME [/var/lib/mysql]</font> 表示在Docker创建MySQL容器时，Docker会自动创建一个volume，将MySQL容器中/var/lib/mysql目录下的内容将同步到这个volume中。

````
docker pull mysql:5.7.29
docker run -d --name mysql-5.7.29 --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=gaoyipeng  mysql:5.7.29
````
然后就可以看到我们已经创建好了mysql-5.7.29，以及创建的卷信息
![mysql-volume.png](/images/docker/mysql-volume.png)

volume 在生成的时候如果不指定名称，便会随机生成（eg:09f037735adf1....）很长不好理解。我们也可以指定名称。
我们删除这个容器，使用如下的命令再次创建，指定volume名称为mysql
````
docker run -d --name mysql-5.7.29 --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=gaoyipeng -v mysql:/var/lib/mysql mysql:5.7.29
````
![volume-mysql.png](/images/docker/volume-mysql.png)
我们使用Navicat链接这个mysql，并创建一个名为docker-volume的数据库。
![navicat-mysql](/images/docker/navicat-mysql.png)
![docker-volume.png](/images/docker/docker-volume.png)

然后我们删除这个容器，使用上面的命令再次创建
```
docker stop mysql-5.7.29
docker rm mysql-5.7.29
docker run -d --name mysql-5.7.29 --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=gaoyipeng -v mysql:/var/lib/mysql mysql:5.7.29
```
使用navacat 再次链接，发现之前创建的docker-volume数据库依然存在，说明数据持久化成功。

## bind mount方式

是直接挂载的宿主机的文件目录，将宿主机的目录映射到容器中；和volume方式的区别就是保存容器数据的宿主机目录需要自行指定，不是docker来管理。

我们修改mysql容器创建命令：
```
docker run -d --name mysql-5.7.29 --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=gaoyipeng -v /usr/local/mysql:/var/lib/mysql mysql:5.7.29 --lower_case_table_names=1
```
这样，容器卷的位置就被指到了宿主机的/usr/local/mysql目录下，而不是/var/lib/docker/volumes。bind mount方式更为灵活。
![bind-volume-mysql.png](/images/docker/bind-volume-mysql.png)

我们可以看到宿主机/usr/local/mysql目录下，生成了持久化文件。

![navicat-no-volume.png](/images/docker/navicat-no-volume.png)

再次使用Navacat连接，发现docker-volume数据库是不存在的，符合预期。

完工！！！

参考：
[https://docs.docker.com/storage/volumes/](https://docs.docker.com/storage/volumes/)
[https://deepzz.com/post/the-docker-volumes-basic.html](https://deepzz.com/post/the-docker-volumes-basic.html)
[https://mrbird.cc/Docker-Volume.html](https://mrbird.cc/Docker-Volume.html)

