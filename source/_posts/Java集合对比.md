---
title: Java集合对比
date: 2019-10-17 11:55:19
tags: [Java,集合]
categories: Java
notebook: 笔记
---

java中集合分List、Set、Map，下面来看看他们的实现类、底层结构及什么情境下使用。

<img src="Java集合对比/cans.jpeg" width="500" height="300"/>

<!-- more -->

# 1.List
## 1.1 ArrayList
底层结构是数组：
- 线程不安全，相对于Vector效率高；
- 查找快，增删慢；

## 1.2 Vector
底层结构是数组：
- 线程安全，相对于ArrayList效率低；
- 查找快，增删慢；

## 1.3 LinkedList
底层结构是链表：
- 线程不安全，效率高；
- 增删快，查找慢；

# 2.Set
## 2.1 HashSet
底层结构是哈希表：
- 无序、唯一；
- 查找快，增删快；

## 2.2 LinkedHashSet
底层结构是链表+哈希表：
- 有序、唯一；
- 查找快，增删快；

## 2.3 TreeSet
底层结构是红黑树：
- 有序、唯一；
- 查找快，增删快；

# 3.Map
## 3.1 HashMap
底层结构是哈希表：
- 线程不安全，相比Hashtable，效率高；
- 无序；
- 查找快，增删快；
- 父类AbstractMap；
- k、v允许为null值

## 3.2 Hashtable
底层结构是哈希表：
- 线程安全，相比HashMap，效率低；
- 无序；
- 查找快，增删快；
- 父类Dictionary；
- k、v不允许为null值；

## 3.3 CurrentHashMap
底层结构是哈希表：
- 线程安全，相比Hashtable做了优化，增删不会锁定整张表，原理是将整个map分成了n个segment；
- 无序；
- 查找快，增删快；

## 3.3 TreeMap
底层结构是红黑树：
- 有序；
- 查找快，增删快；


- - -
<b>One is never too old to learn.</b>