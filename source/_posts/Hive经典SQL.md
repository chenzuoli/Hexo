---
title: Hive经典SQL
date: 2019-10-25 10:04:00
tags: [Hive,SQL]
categories: Hive
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面来看下数据仓库数据分析过程中使用到比较经典的Hive SQL的使用。

<img src="Hive经典SQL/hive.jpeg" width="500" height="300"/>

<!-- more -->

1. 有十万个淘宝店铺，每个顾客访问任意一个店铺时都会生成一条访问日志，访问日志表为visit，其中uid为用户id，store为店铺名称，统计店铺的uv；
```
select store, count(distinct uid) from visit group by store;
```

2. 有一亿个用户存储在user表中，有字段uid，age（年龄），total_consume（消费总金额），使用hive sql或者spark sql按照用户年龄大小降序排序，如果年龄相同按照消费总金额升序排列；
```
select uid, age, total_consume from `user` order by age desc, total_consume;
```


- - -
<b>Just do it.</b>