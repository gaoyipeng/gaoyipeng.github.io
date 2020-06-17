---
title: Docker网络配置笔记
date: 2020-01-27 10:12:59
tags: Docker
categories: Docker
description: Docker网络配置笔记
typora-root-url: ..
---

## docker网络模式
Docker自身拥有4种默认的网络模式，另外我们还可以自定义网络。

在安装Docker时，会在宿主机上创建一个新的网络接口，名字是docker0。docker0是一个虚拟的以太网桥，它专门用于连接容器和本地宿主机网络.
每个Docker容器都会在这个接口上分配一个IP地址，接口本身的地址是172.17.0.1，子网掩码是255.255.0.0，也就是说Docker会默认使用172.17.X.X作为子网地址。

![docker0](/images/docker/docker0.png)

### 四种默认网络模式

1. bridge模式（默认模式）：使用\-\-net=bridge指定

   此模式会为每一个容器分配、设置IP等，并将容器连接到一个docker0虚拟网桥，通过docker0网桥以及Iptables nat表配置与宿主机通信。
2. Host模式：使用\-\-net=host指定

   容器和宿主机共享Network namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。使用host模式的容器可以直接使用宿主机的IP地址与外界通信，容器内部的服务端口也可以使用宿主机的端口，不需要进行NAT。
   
   host模式的优势就是网络性能比较好，但是docker host上已经使用的端口就不能再用了，网络的隔离性不好。
3. Container模式：使用\-\-net=container:NAME_or_ID指定

   容器和另外一个容器共享Network namespace。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。
4. None模式（默认模式）：使用\-\-net=none指定

   容器有独立的Network namespace，但并没有对其进行任何网络设置，如分配veth pair 和网桥连接，配置IP等。
   
   注意：这种类型的网络没有办法联网，封闭的网络能很好的保证容器的安全性。

## bridge模式（重点）
    我们先来创建2个busybox容器。
    docker run -itd --name busybox1 busybox
    docker run -itd --name busybox2 busybox

我们使用 docker network inspect bridge 可以查看bridge网络的详情
![bridge详情.png](/images/docker/bridge详情.png)

可以看到容器名称，IP地址，Container Id 信息。

我们进入busybox1容器

    docker exec -it busybox1 sh

我们看下在 busybox1 容器内部是否可以ping通busybox2的IP。再尝试下是否可以ping通busybox2的 Container Name
![ping-ip-busybox2.png](/images/docker/ping-ip-busybox2.png)

我们发现busybox1和busybox2可以通过IP互通，但是无法通知container name通讯。所以默认bridge只支持ip互通。

### \-\- link

如果我们想让bridge模式的Container之间支持通过Container Name通讯。我们可以使用\-\-link 的方式。
我们删除busybox2容器。重新创建busybox2容器，并且让其和busybox1建立网络映射关系。
    
    docker stop busybox2
    docker rm busybox2
    docker run -itd --name busybox2 --link busybox1 busybox

![busybox2-ping-busybox1.png](/images/docker/busybox2-ping-busybox1.png)

busybox2 容器内部可以通过container name ping通busybox1
然而反过来busybox1 容器内部 **不可以** 通过 container name ping通busybox2。

我们删除busybox1容器。重新创建busybox1容器，并且让其和busybox2建立网络映射关系。

    docker stop busybox1
    docker rm busybox1
    docker run -itd --name busybox1 --link busybox2 busybox

![busybox1-ping-busybox2.png](/images/docker/busybox1-ping-busybox2.png)

这样就可以了.


总结一下: \-\-link方式可以在容器间建立映射关系。如果想容器间互相通过container name 通信，则容器间都需要 \-\-link 链接。

## 自定义网络（重点）

除了\-\-link 可以实现容器间 container name 通信，还有自定义网桥这个方式。

命令：

    docker network create -d <bridge|host|container|none> network_name

下面我们命名一个my-bridge的自定义网络。通常情况下使用如下命令即可创建。

    docker network create -d bridge my-bridge

模式选择的是默认的bridge。

使用docker network ls即可查看网络列表
![my-bridge.png](/images/docker/my-bridge.png)

使用docker run 创建容器时，添加\-\-net=my-bridge即可配置使用自定义网络。

如果是已经创建好的容器，通过以下命令可以连接到自定义网络上：
    docker network connect my-bridge busybox1
![my-bridge-box1.png](/images/docker/my-bridge-box1.png)


那么如何在docker-compose.yml中配置自定义网络呢？
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
    
    ......
    networks:
      - my-bridge  
networks:
  my-bridge:
    driver: bridge
```

以上docker-compose.yml同样可以创建my-bridge自定义网络。并且创建的elasticsearch容器会配置使用my-bridge自定义网络。
![docker-network.png](/images/docker/docker-network.png)

我们发现自动给my-bridge添加了elk前缀，变为了elk_my-bridge ，elk是docker-compose.yml文件的目录名。感觉是为了避免重名吧，不太清楚了！！

我们查看下elk_my-bridge详情
    docker network inspect elk_my-bridge 
```
docker network inspect elk_my-bridge 
[
    {
        "Name": "elk_my-bridge",
        "Id": "d28de6a1cda8b97a282bdab80a10eeae6ac75acb27c3992f51799dad49f0c98c",
        "Created": "2020-01-27T22:10:46.179450229+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "4dbffe4469778db7574198a4336b3637653c434a541c102728b9dcf321824c85": {
                "Name": "kibana",
                "EndpointID": "1dc354be523cc7cbb944545be273d4b0c0b85e0e9b00463213b412a7879a7390",
                "MacAddress": "02:42:ac:13:00:04",
                "IPv4Address": "172.19.0.4/16",
                "IPv6Address": ""
            },
            "71607bb793ed6ee0faf9e9d26967c8588abc8076a2cdf6cf1822fb0f321c2c0c": {
                "Name": "elasticsearch",
                "EndpointID": "e873552071bd5cf9dd86bc564be568ff37ca74ab51c59d1a68d8b416b1cb0da3",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "e5c7736099fe779d7718f35557161a05ba2ba03ddcbf20c73a4146794fbb7df8": {
                "Name": "logstash",
                "EndpointID": "5e8821f7e67780468664b7947be64fdb0bf2de3e8b2b2f73d675dd8e29ee0878",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "my-bridge",
            "com.docker.compose.project": "elk",
            "com.docker.compose.version": "1.22.0"
        }
    }
]

```

可以看到elk_my-bridge网桥的详情，而且发现容器的IP变为了172.19.X.X不再是默认docker 网段172.17.X.X。应该是自定义网桥的原因吧。
elasticsearch，logstash，kibana之间是可以通过容器名来通信的。

OK，今天就到这吧，2020年春节，冠状病毒肺炎疫情很严峻，闷在家里总结下以前的知识点。愿这次的疫情尽早解决吧，武汉加油！湖北加油！！中国加油！！！

参考：
https://www.cnblogs.com/kaye/p/10508800.html
https://blog.csdn.net/wucong60/article/details/83757813
https://www.cnblogs.com/zuxing/articles/8780661.html
https://www.jianshu.com/p/22a7032bb7bd
https://mrbird.cc/Docker-network.html