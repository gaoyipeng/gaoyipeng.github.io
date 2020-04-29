---
title: SQL 调优
date: 2020-03-01 16:37:15
tags: [数据库,Mysql,sql调优]
categories: [数据库,Mysql]
description: SQL 调优
---

## SQL优化方式

### 小表驱动大表

![sql-exist.png](/images/database/sql-exist.png)

### Order by 优化

* Order by 语句应该符合最佳左前缀原则。尽可能在索引列上完成排序
eg:
索引：index  idx_n_a(name,age)
select name,age from user age>20 order by name,age (√)
select name,age from user age>20 order by name     (√)
select name,age from user age>20 order by age      (×)
select name,age from user age>20 order by age,name (×)

注意，当索引idx_n_a的第一个列 name为常量时，order by age也可以使用索引
select name,age from user age=20 order by age      (√)

* 使用Order by 时最好不要使用select * ，使用select colume1,colume2,....from table Order by colume1，colume2...
* 使用Order by 查询慢，可以尝试提高系统参数 sort_buffer_size 的值
* 使用Order by 查询慢，可以尝试提高系统参数 max_length_for_sort_data 的值

### group by 优化

与order by 的优化方式基本一致。主要注意的点：

* group by先排序后分组，符合最佳左前缀原则
* 能使用where限定条件，就不要使用having

Having与Where的区别
1. where 子句的作用是在对查询结果进行分组前，将不符合where条件的行去掉，即在分组之前过滤数据，where条件中不能包含聚组函数，使用where条件过滤出特定的行。
2. having 子句的作用是筛选满足条件的组，即在分组之后过滤数据，条件中经常包含聚组函数，使用having 条件过滤出特定的组，也可以使用多个分组标准进行分组。

### 开启慢查询日志

慢查询日志默认不开启(影响性能，需要时开启即可)：
```
mysql> show variables like '%slow_query_log%';
+---------------------+--------------------------+
| Variable_name       | Value                    |
+---------------------+--------------------------+
| slow_query_log      | ON                       |
| slow_query_log_file | WIN-03C6PBEDETT-slow.log |
+---------------------+--------------------------+
2 rows in set (0.06 sec)
```

开启命令(只对当前数据库有效，重启数据库则失效)：
```
mysql> set global slow_query_log=1;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like '%slow_query_log%';
+---------------------+--------------------------+
| Variable_name       | Value                    |
+---------------------+--------------------------+
| slow_query_log      | ON                       |
| slow_query_log_file | WIN-03C6PBEDETT-slow.log |
+---------------------+--------------------------+
2 rows in set (0.01 sec)

mysql> show variables like '%long_query_time%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.01 sec)
```

上面我们开启了慢查询日志，把查询时间超过10秒的记录到WIN-03C6PBEDETT-slow.log(这是windows环境，Linux下slow_query_log_file会指明慢查询文件路径)中。
我们也可以修改 long_query_time 值
```
set global long_query_time=3;
```

我们可以使用mysqldumpslow 分析日志文件。语法：mysqldumpslow 参数 日志文件。
eg ：
```
mysqldumpslow -s r -t 10 WIN-03C6PBEDETT-slow.log
```
参数列表：
![mysqldumpslow.png](/images/database/mysqldumpslow.png)

### Show Profile sql分析
开启功能
```
mysql> show variables like '%profiling%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| have_profiling         | YES   |
| profiling              | OFF   |
| profiling_history_size | 15    |
+------------------------+-------+
3 rows in set (0.02 sec)

mysql> set profiling=1;
Query OK, 0 rows affected (0.01 sec)

mysql> show variables like '%profiling%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| have_profiling         | YES   |
| profiling              | ON    |
| profiling_history_size | 15    |
+------------------------+-------+
3 rows in set (0.03 sec)
```

查看使用过的SQL,其中Duration显示执行时间
```
mysql> show profiles;
+----------+------------+-----------------------------------+
| Query_ID | Duration   | Query                             |
+----------+------------+-----------------------------------+
|        1 | 0.00210800 | show variables like '%profiling%' |
|        2 | 0.00064725 | select * from user                |
|        3 | 0.01892075 | select * from wmi                 |
+----------+------------+-----------------------------------+
3 rows in set (0.02 sec)
```

查看SQL执行过程，语法：
```
show profile <type> for query <Query_ID>;
```

可用type参数列表
![show-profiles-type.png](/images/database/show-profiles-type.png)

eg:查看Query_ID为3的SQL的cpu及io耗时
```
mysql> show profile cpu,block io for query 3;
+----------------------+----------+----------+------------+--------------+---------------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+----------------------+----------+----------+------------+--------------+---------------+
| starting             | 0.000049 | 0.000000 | 0.000000   | NULL         | NULL          |
| checking permissions | 0.000025 | 0.000000 | 0.000000   | NULL         | NULL          |
| Opening tables       | 0.000022 | 0.000000 | 0.000000   | NULL         | NULL          |
| init                 | 0.000062 | 0.000000 | 0.000000   | NULL         | NULL          |
| System lock          | 0.000033 | 0.000000 | 0.000000   | NULL         | NULL          |
| optimizing           | 0.000005 | 0.000000 | 0.000000   | NULL         | NULL          |
| statistics           | 0.000016 | 0.000000 | 0.000000   | NULL         | NULL          |
| preparing            | 0.000012 | 0.000000 | 0.000000   | NULL         | NULL          |
| executing            | 0.000003 | 0.000000 | 0.000000   | NULL         | NULL          |
| Sending data         | 0.018619 | 0.015625 | 0.000000   | NULL         | NULL          |
| end                  | 0.000008 | 0.000000 | 0.000000   | NULL         | NULL          |
| query end            | 0.000007 | 0.000000 | 0.000000   | NULL         | NULL          |
| closing tables       | 0.000008 | 0.000000 | 0.000000   | NULL         | NULL          |
| freeing items        | 0.000025 | 0.000000 | 0.000000   | NULL         | NULL          |
| cleaning up          | 0.000030 | 0.000000 | 0.000000   | NULL         | NULL          |
+----------------------+----------+----------+------------+--------------+---------------+
15 rows in set (0.03 sec)
```
我们需要注意Status列是否会出现如下内容：

![show-profiles-status.png](/images/database/show-profiles-status.png)