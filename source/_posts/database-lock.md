---
title: 数据库锁机制
date: 2020-03-01 19:38:42
tags: [数据库,Mysql]
categories: [数据库,Mysql]
description: 数据库锁机制
typora-root-url: ..
---

## 数据库引擎

mysql常见的数据库引擎有MyISAM、InnoDB。
MySQL中myisam与innodb的区别：

* InnoDB支持事物，而MyISAM不支持事物
* InnoDB支持行级锁，而MyISAM支持表级锁
* InnoDB支持外键，而MyISAM不支持
* InnoDB提供提交、回滚、崩溃恢复能力的事务安全（ASID）能力，实现并发控制。
* MyISAM提供较高的插入和查询记录的效率，主要用于插入和查询。

## 数据库锁分类

### 乐观锁和悲观锁概念

悲观锁：总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。
乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。

共享锁和排他锁都是悲观锁的实现方式，由数据库提供。乐观锁就需要自己实现了。


### 操作粒度分类

* 表锁 ：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。
* 行锁 ：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
* 页锁（不常见） ：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般


#### 表锁

1. 查看数据库每张表使用表锁的情况：
```
mysql> show open tables;
+--------------------+------------------------------------------------------+--------+-------------+
| Database           | Table                                                | In_use | Name_locked |
+--------------------+------------------------------------------------------+--------+-------------+

| vim-base           | user_role_bak_190902                                 |      0 |           0 |
| performance_schema | status_by_user                                       |      0 |           0 |
| febs_cloud_base    | zipkin_spans                                         |      0 |           0 |
......
| vim-wmi            | vin_group                                            |      0 |           0 |
| vim-base           | enterprise_update                                    |      0 |           0 |
| vim-wmi            | wmi_detail                                           |      0 |           0 |
| performance_schema | events_stages_summary_by_host_by_event_name          |      0 |           0 |
+--------------------+------------------------------------------------------+--------+-------------+
323 rows in set (0.20 sec)
```
In_use 为1 代表使用了锁。

2. 表锁命令：
* 加锁 lock table \<tableName\> read\|write;
* 解锁 unlock tables;

3. 以下是MyISAM表锁的总结：
![table-lock.png](/images/database/table-lock.png)

4. 分析数据库表锁定
```
mysql> show status like 'table%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Table_locks_immediate      | 340   |
| Table_locks_waited         | 0     |
| Table_open_cache_hits      | 22    |
| Table_open_cache_misses    | 1     |
| Table_open_cache_overflows | 0     |
+----------------------------+-------+
5 rows in set (0.02 sec)
```

* Table_locks_immediate：产生表级锁定的次数，表示可以立即获取锁的次数。
* Table_locks_waited：产生表级锁定争用二发生等待的次数，次数愈多，表明表级锁争用情况越严重。

#### 行锁

Innodb 引擎，默认是行锁。

注意：行锁是依赖索引的，如果没有索引或者索引失效，则会自动转为表锁。

1. 间隙锁
![next-key.png](/images/database/next-key.png)

### 操作类型分类

#### 共享锁（S锁、读锁）
对同一数据，多个读操作可同时进行。但是会阻塞其他进程的写操作。

#### 排他锁（X锁、写锁）
当前写操作没完成前，阻断其他进程的读和写操作。

1. 查看数据库表锁情况
```
mysql> show status like 'innodb_row_lock%';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0     |
| Innodb_row_lock_time          | 2     |
| Innodb_row_lock_time_avg      | 2     |
| Innodb_row_lock_time_max      | 2     |
| Innodb_row_lock_waits         | 1     |
+-------------------------------+-------+
5 rows in set (0.02 sec)
```
![hang-lock.png](/images/database/hang-lock.png)

2. 语法：
共享锁：SQL+ lock in share mode;
排它锁：SQL+ for update;

### 死锁
所谓死锁：是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。

产生死锁的四个必要条件：

* 互斥条件：一个资源每次只能被一个进程使用。
* 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
* 不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
* 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

这四个条件是死锁的必要条件，只要系统发生死锁，这些条件必然成立，而只要上述条件之一不满足，就不会发生死锁。