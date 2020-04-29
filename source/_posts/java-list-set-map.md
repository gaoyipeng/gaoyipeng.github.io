---
title: Java中的List、Set和Map的各自特征
date: 2020-03-25 22:32:20
tags: [Java,数据结构]
categories: [Java,数据结构]
description: Java中的List、Set和Map的各自特征
---

先通过这张图看看Collection和Map的各自体系
![list-set-map.png](/images/Java/list-set-map.png)


## List 

* 有序（指存入元素的先后顺序与输出元素的先后顺序一致）
* 元素可重复
* 可以插入null, 数量不限

### ArrayList

* 底层数据结构是数组,查询快,增删慢  
* 线程不安全,效率高

### LinkedList

* 底层数据结构是链表,查询慢,增删快
* 线程不安全,效率高

### Vectory

* 底层数据结构是数组,查询快,增删慢 
* 线程安全,效率低

## Set

* 无序（指存入元素的先后顺序与输出元素的先后顺序不一致）
* 元素不可重复
* 可以插入null, 且只能插入一个

### HashSet

* 内部的数据结构是哈希表
* 线程不安全的。
* HashSet中保证集合中元素是唯一的方法：通过对象的hashCode和equals方法来完成对象唯一性的判断。

#### LinkedHashSet

* 底层数据结构由链表和哈希表组成
* 线程不安全的
* LinkedHashSet是一种有序的Set集合，即其元素的存入和输出的顺序是相同的

### SortedSet(接口)

#### TreeSet

* 底层数据结构是红黑树
* 可以对Set集合中的元素进行排序
    1. 自然排序(元素具备比较性):需要实现Comparable接口，并覆盖其compareTo方法。
    2. 比较器排序(元素不具备比较性):则需要实现Comparator接口，并覆盖其compare方法

## Map

键值对形式存在，Map集合的数据结构仅仅针对键有效，与值无关。

### HashMap (未完，后续继续补充)
底层结构是数组+链表，jdk1.8后还引入和红黑树。