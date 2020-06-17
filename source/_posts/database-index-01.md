---
title: 数据库索引
date: 2020-02-27 00:08:15
tags: [数据库,Mysql,面试]
categories: [数据库,Mysql]
description: 数据库索引
typora-root-url: ..
---

{% note info %}
MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构。即索引的本质：索引是数据结构。
但方法都是有利有弊，索引在方便你查询的时候，也要牺牲磁盘的空间为代价。
数据库索引可以协助快速查询,更新数据库中表的数据.索引的实现通常使用B树和变种的B+树(mysql常用的索引就是B+树){% endnote %}

## 索引结构分类

Mysql目前主要有以下几种索引结构：HASH，BTREE，RTREE

* BTREE
BTREE索引就是一种将索引值按一定的算法，存入一个树形的数据结构中（二叉树），每次查询都是从树的入口root开始，依次遍历node，获取leaf。
<label style="color:red">BTREE是MySQL里默认和最常用的索引类型</label>

* HASH
由于HASH的唯一（几乎100%的唯一）及类似键值对的形式，很适合作为索引。
HASH索引可以一次定位，不需要像树形索引那样逐层查找,因此具有极高的效率。但是，这种高效是有条件的，即只在“=”和“in”条件下高效，对于范围查询、排序及组合索引仍然效率不高。

* RTREE
RTREE在MySQL很少使用，仅支持geometry数据类型，支持该类型的存储引擎只有MyISAM、BDb、InnoDb、NDb、Archive几种。
相对于BTREE，RTREE的优势在于范围查找。

## 索引类型

mysql 索引有2种分类标准
1. 从物理存储角度分为：聚簇索引、非聚簇索引
2. 从逻辑存储角度分为：普通索引、唯一索引、主键索引、组合索引、全文索引

### 聚簇索引和非聚簇索引

* 聚簇索引：是对磁盘上实际数据重新组织以按指定的一个或多个列的值排序的算法。特点是存储数据的顺序和索引顺序一致。
<label style="color:red">一个表只能有一个聚簇索引。主键索引一般都是聚簇索引</label>

* 非聚簇索引：表数据存储顺序与索引顺序无关。对于非聚簇索引，叶结点包含索引字段值及指向数据页数据行的逻辑指针，其行数量与数据表行数据量一致。非聚簇索引记录的物理顺序与逻辑顺序没有必然的联系，与数据的存储物理结构没有关系；一个表对应的非聚簇索引可以有多条，根据不同列的约束可以建立不同要求的非聚簇索引；

建立聚簇索引的语句：
```
CREATE CLUSTER INDEX index_name ON table_name(column_name1,...);
```

#### 聚簇索引 VS 非聚簇索引

* 聚簇索引的叶子节点就是数据节点（Innodb的B+树的主键对应的数据节点），非聚簇索引的叶子节点仍然是索引节点，只不过有指向对应数据块的指针。
* 聚簇索引主键的插入速度要比非聚簇索引主键的插入速度慢很多。聚簇索引适合排序，非聚簇索引不适合用在排序的场合。因为聚簇索引本身已经是按照物理顺序放置的，排序很快。非聚簇索引则没有按序存放，需要额外消耗资源来排序。

### 普通索引、唯一索引、主键索引、组合索引、全文索引

#### 普通索引（NORMAL）
> 仅加速查询,无限制

#### 唯一索引（UNIQUE）
> 加速查询 + 列值唯一（可以有null）,如果是组合索引，则列值的组合必须唯一

#### 主键索引 (PRIMARY KEY)
> 加速查询 + 列值唯一（不可以有null）+ 表中只允许有一个主键索引, 是一种特殊的唯一索引

#### 组合索引
> 多列值组成一个索引，专门用于组合搜索，其效率大于索引合并（索引合并：使用多个单列索引组合搜索）
> 只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循最左前缀集合

eg: 我们新建一张表：
```
CREATE TABLE mytable( ID INT NOT NULL, username VARCHAR(16) NOT NULL, city VARCHAR(50) NOT NULL, age INT NOT NULL );
```
为了进一步榨取MySQL的效率，就要考虑建立组合索引。就是将 name, city, age建到一个索引里：
```
ALTER TABLE mytable ADD INDEX name_city_age (name(10),city,age);
```
建表时，usernname长度为 16，这里用 10。这是因为一般情况下名字的长度不会超过10，这样会加速索引查询速度，还会减少索引文件的大小，提高INSERT的更新速度。
如果分别在 usernname，city，age上建立单列索引，让该表有3个单列索引，查询时和上述的组合索引效率也会大不一样，远远低于我们的组合索引。虽然此时有了三个索引，但MySQL只能用到其中的那个它认为似乎是最有效率的单列索引。

建立这样的组合索引，其实是相当于分别建立了下面三组组合MySQL数据库索引：
1. usernname,city,age
2. usernname,city
3. usernname 

为什么没有 city，age这样的组合索引呢？这是因为MySQL组合索引“最左前缀”的结果。简单的理解就是只从最左面的开始组合。并不是只要包含这三列的查询都会用到该组合索引，下面的几个SQL就会用到这个组合MySQL数据库索引：
```
SELECT * FROM mytable WHREE username="admin" AND city="郑州"
SELECT * FROM mytable WHREE username="admin"
```
而下面几个则不会用到：

```
SELECT * FROM mytable WHREE age=20 AND city="郑州" 
SELECT * FROM mytable WHREE city="郑州"
```


#### 全文索引 (FULLTEXT)
> 它能够利用【分词技术】等多种算法智能分析出文本文字中关键词的频率和重要性，然后按照一定的算法规则智能地筛选出我们想要的搜索结果。

全文索引，以前只有MyISAM引擎支持。Mysql5.6后开始默认使用InnoDb引擎，并且5.6.4版本后的InnoDb引擎开始支持全文索引。 
目前只有 CHAR、VARCHAR 、TEXT 列上可以创建全文索引。

select * from article where content like '%xxx%';这种写法查询效率非常低。全文索引就是为了高效的解决这种问题产生的。

具体如何使用全文索引呢？

不用全文索引时的写法：
```
SELECT * FROM article WHERE content LIKE '%查询字符串%';
```

如果我们在content字段建立了全文索引，那么使用全文索引的写法：
```
SELECT * FROM article WHERE MATCH(title,content) AGAINST ('查询字符串');
```

注意：在数据量较大时候，我们应该先将数据放入一个没有全局索引的表中，然后再用CREATE index创建fulltext索引，要比先为一张表建立fulltext然后再将数据写入的查询速度快。

#### 覆盖索引
> select的数据列只用从索引中就能够取得，不必读取数据行，换句话说查询列要被所建的索引覆盖

例如：
表 user 中有一个组合索引 idx_name_sex(name,sex)。
当我们通过SQL语句：select name from user where name = 'garnett';的时候，就可以通过覆盖索引查询.即查询条件包含在组合索引的列中，换句话说查询的列被索引覆盖。


## 规则

### 需要建立索引的情况

* 主键自动建立主键索引 (PRIMARY KEY)
* 频繁作为查询条件的字段
* 与其他表关联的字段，外键关系创建索引。（eg：user表中的dept_id）
* 查询中统计或者分组的字段

### 不适合建立索引的情况

* 表记录太少就别建索引了
* 频繁更新的字段不要创建索引
* 数据列重复的字段不需要创建索引，没必要。（eg：性别字段，不是男就是女）
* 对于那些在查询中很少使用或者参考的列不应该创建索引

### join查询索引建立规则

* 左连接：右表关联字段建索引
* 右连接：左表关联字段建索引

## 索引失效情况

* 条件中有or，即使其中有条件带索引也不会使用(这也是为什么尽量少用or的原因)。注意：要想使用or，又想让索引生效，只能将or条件中的每个列都加上索引
* 在索引字段上使用计算、函数等，会导致索引失效。例如left(name,4),导致name字段索引失效。
* 对于组合索引，如果不使用的组合索引的第一列字段，则不会使用索引（即不符合最左前缀原则）
* 如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引
* 范围查找，右边的索引全失效。eg:一个组合索引 index（name,age.sex）
``
select name,age,sex from where name= 'aa' and age >25 and sex=1 
``
sex 字段上的索引，无法被使用

* 使用 != 和 <> 会导致无法使用索引，变为全表扫描。
* is null 和 is not null 无法使用索引
* like '%aa',以通配符开头，索引会失效。针对这种情况，我们可以使用覆盖索引解决。
 eg: 创建一个组合索引 index（name,age），此时使用 select name from user where name like '%aa%',索引不会失效


## 查看索引的使用情况

```
show status like ‘Handler_read%’;
```

![handler_read.png](/images/database/handler_read.png)

* handler_read_key:这个值越高越好，越高表示使用索引查询到的次数越多
* handler_read_rnd_next:这个值越高越不好，说明查询低效

参考文章：
https://www.cnblogs.com/LiLiliang/p/9960895.html
https://blog.csdn.net/weixin_34379433/article/details/89745873
https://blog.csdn.net/qq_36071795/article/details/83956068