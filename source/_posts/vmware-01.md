---
title: 解决CentOS7克隆虚拟机，ifconfig命令不显示ip地址问题
date: 2020-03-12 12:29:39
tags: VMware
categories: VMware
description: 解决CentOS7克隆虚拟机，ifconfig命令不显示ip地址问题
typora-root-url: ..
---

## 事故起因

昨晚遇到个问题，我本机开了3个centos7 虚拟机，都是使用VMware克隆出来的。然后，电脑卡死~

重启电脑后，再次打开虚拟机，发现Xsheel无法连接了，进入虚拟机使用ipconfig查看ip地址，发现：
```
[root@localhost admin]# ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:abff:fe00:ce0c  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ab:00:ce:0c  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 29  bytes 3240 (3.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 12  bytes 1068 (1.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12  bytes 1068 (1.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethe27fbd4: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::10ea:7bff:feb5:a3d7  prefixlen 64  scopeid 0x20<link>
        ether 12:ea:7b:b5:a3:d7  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 37  bytes 3888 (3.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:80:2c:25  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@localhost admin]# 
```

可以看到不显示eth33.这样我们就无法得知虚拟机ip。

## 解决方案

### 重建适配器:在vmware虚拟机中执行
```
systemctl stop NetworkManager
systemctl disable NetworkManager
```
### 然后，关闭虚拟机（不然点击不了生成）
### 设置虚拟机

依次点击：
![setting.png](/images/vmware/setting.png)

再次使用ifconfig查看就可以看到eth33下的ip地址了。


参考：https://blog.csdn.net/qq_36434219/article/details/80671978