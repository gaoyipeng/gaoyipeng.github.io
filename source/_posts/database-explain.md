---
title: 使用explain对sql语句进行性能分析
date: 2020-02-29 20:21:19
tags: [数据库,Mysql]
categories: [数据库,Mysql]
description: 使用explain对sql语句进行性能分析
typora-root-url: ..
---
{% note info %}在日常工作中，我们有时会遇到一些慢查询的情况，些时我们常常用到explain这个命令来查看一个这些SQL语句的执行计划，查看该SQL语句有没有使用上了索引，有没有做全表扫描等。

这都可以通过explain命令来查看。所以我们深入了解MySQL的基于开销的优化器，还可以获得很多可能被优化器考虑到的访问策略的细节，以及当运行SQL语句时哪种策略预计会被优化器采用。
{% endnote %}

## 可以干嘛

使用explain分析sql,我们可以分析出以下结果：

* 表的读取顺序
* 数据读取操作的操作类型
* 哪些索引可以使用
* 哪些索引被实际使用
* 表之间的引用
* 每张表有多少行被优化器查询

## 环境说明

```
mysql> select version();
+------------+
| version()  |
+------------+
| 5.7.17-log |
+------------+
1 row in set (0.02 sec)
```

## 怎么用

格式：explain + sql （命令行界面）

我们来查看一个explain的使用示例(正好写了一个sql,拿来分析一下)：

```
mysql> explain SELECT
 w.id 'ID',
 w.application_code '申请号码',
 w.enterprise_id '企业ID',
 w.enterprise_name '企业名称',
 w.wmi_code 'wmi编码',
 w.vin_id 'VINID',
 w.`status` '状态',
 w.origin_id '原始ID',
 w.type '类型',
 w.certificate_no '证书编号',
 d.certificate_start '原证书开始时间',
 d.certificate_end '原证书结束时间',
 w.certificate_start '开始时间',
 w.certificate_end '结束时间',
 w.deferred_state '证书延期状态',
 w.created_at '创建时间',
 w.update_at '修改时间',
 w.submitted_at '提交时间',
 w.created_by '创建人ID' 
FROM
 `vim-wmi`.wmi w ,
  (SELECT * FROM ( SELECT id,origin_id,certificate_start,certificate_end FROM `vim-wmi`.wmi WHERE `status` = 12 ) b) d
WHERE
 w.origin_id IN ( SELECT * FROM ( SELECT id FROM `vim-wmi`.wmi WHERE `status` = 12 ) a ) 
 AND w.`status`  IN (9,12 ) and w.origin_id = d.id AND w.type = 383;
+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref                 | rows | filtered | Extra       |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------------+
|  1 | SIMPLE      | w     | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                | 2756 |     2.00 | Using where |
|  1 | SIMPLE      | wmi   | NULL       | eq_ref | PRIMARY       | PRIMARY | 128     | vim-wmi.w.origin_id |    1 |    10.00 | Using where |
|  1 | SIMPLE      | wmi   | NULL       | eq_ref | PRIMARY       | PRIMARY | 128     | vim-wmi.w.origin_id |    1 |    10.00 | Using where |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------------+
3 rows in set (0.05 sec)

```
expain 分析出来的信息分别是id、select_type、table、partitions、type、possible_keys、key、key_len、ref、rows、filtered、Extra

### id

select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序

id的结果共有3种排序情况：
1. id 相同，语句执行顺序：顺序执行
![explain_id_1.png](/images/database/explain_id_1.png)

加载表的顺序如上图table列所示：t1 t2 t3

2. id 值都不同，id的序号会递增，id值越大优先级越高，越先被执行
![explain_id_1.png](/images/database/explain_id_2.png)

加载表的顺序如上图table列所示：t3 t1 t2

3、id 值有的相同有的不同，id值越大优先级越高，相同值顺序执行
![explain_id_1.png](/images/database/explain_id_3.png)

加载表的顺序如上图table列所示：t3 t2 <derived2>，<derived2>是一个衍生表，2即是代表id为2的那一行。

### select_type
分别用来表示查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询。

分为如下6种
* SIMPLE：简单的select查询，查询中不包含子查询或者UNION
* PRIMARY：查询中若包含任何复杂的子部分，最外层查询则被标记为PRIMARY
* SUBQUERY：在SELECT或WHERE列表中包含了子查询
* DERIVED：在FROM列表中包含的子查询被标记为DERIVED（衍生），MySQL会递归执行这些子查询，把结果放在临时表中
* UNION：若第二个SELECT出现在UNION之后，则被标记为UNION：若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
* UNION RESULT： 从UNION表获取结果的SELECT

### table 

显示的查询表名，如果查询使用了别名，那么这里显示的是别名，如果不涉及对数据表的操作，那么这显示为null。
如果显示为尖括号括起来的<derived N>就表示这个是临时表，后边的N就是执行计划中的id，表示结果来自于这个查询产生。

### partitions

对分区的说明

### type

type所显示的是查询使用了哪种类型

从最好到最差依次是：
```
system，const，eq_ref，ref，fulltext，ref_or_null，index_merge，unique_subquery，index_subquery，range，index，ALL

```
一般来说，得保证查询至少达到range级别，最好能达到ref。

介绍几个经常见到的：
* system 表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计
* const 表示通过索引一次就找到了,const用于比较primary key 或者unique索引。因为只匹配一行数据，所以很快。
* eq_ref 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描.
* ref 非唯一性索引扫描，我理解为用过索引找到了符合查询条件的多条结果
* range 只检索给定范围的行，一般就是在你的where语句中出现between、< 、>、in等的查询，这种范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引。
* index Index与All区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。（全索引遍历）
* all 将遍历全表以找到匹配的行

### possible_keys 和 key

* possible_keys：显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用。
* key：实际使用的索引，如果为NULL，则没有使用索引。（可能原因包括没有建立索引或索引失效），查询中若使用了覆盖索引（select 后要查询的字段刚好和创建的索引字段完全相同），则该索引仅出现在key列表中

### key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度，在不损失精确性的情况下，长度越短越好。
key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的。

同样查询结果，key_len越小越好。

### ref 

显示索引的那一列被使用了，或哪一个常量被用于查找索引列上的值。

### rows

这里是执行计划中估算的扫描行数，不是精确值。越少越好

### filtered
指返回结果的行占需要读到的行(rows列的值)的百分比

### Extra

这里显示些其他的信息。要注意extra辅助信息中的using filesort和using temporary，这两项非常消耗性能，需要注意。

这个列可以显示的信息非常多，有几十种，常用的有：
* distinct：在select部分使用了distinc关键字
* no tables used：不带from字句的查询或者From dual查询
* 使用not in()形式子查询或not exists运算符的连接查询，这种叫做反连接。即，一般连接查询是先查询内表，再查询外表，反连接就是先查询外表，再查询内表。
* using filesort：排序时无法使用到索引时，就会出现这个。常见于order by和group by语句中
* using index：查询时不需要回表查询，直接通过索引就可以获取查询的数据。（说明效率高）
* using join buffer（block nested loop），using join buffer（batched key accss）：5.6.x之后的版本优化关联查询的BNL，BKA特性。主要是减少内表的循环数量以及比较顺序地扫描查询。
* using sort_union，using_union，using intersect，using sort_intersection：
* using intersect：表示使用and的各个索引的条件时，该信息表示是从处理结果获取交集
* using union：表示使用or连接各个使用索引的条件时，该信息表示从处理结果获取并集
* using sort_union和using sort_intersection：与前面两个对应的类似，只是他们是出现在用and和or查询信息量大时，先查询主键，然后进行排序合并后，才能读取记录并返回。
* using temporary：表示使用了临时表存储中间结果。临时表可以是内存临时表和磁盘临时表，执行计划中看不出来，需要查看status变量，used_tmp_table，used_tmp_disk_table才能看出来。
* using where：表示存储引擎返回的记录并不是所有的都满足查询条件，需要在server层进行过滤。查询条件中分为限制条件和检查条件，5.6之前，存储引擎只能根据限制条件扫描数据并返回，然后server层根据检查条件进行过滤再返回真正符合查询的数据。5.6.x之后支持ICP特性，可以把检查条件也下推到存储引擎层，不符合检查条件和限制条件的数据，直接不读取，这样就大大减少了存储引擎扫描的记录数量。extra列显示using index condition
* firstmatch(tb_name)：5.6.x开始引入的优化子查询的新特性之一，常见于where字句含有in()类型的子查询。如果内表的数据量比较大，就可能出现这个
* loosescan(m..n)：5.6.x之后引入的优化子查询的新特性之一，在in()类型的子查询中，子查询返回的可能有重复记录时，就可能出现这个

除了这些之外，还有很多查询数据字典库，执行计划过程中就发现不可能存在结果的一些提示信息

