---
title: Presto和Hive的对比
date: 2019-10-14 21:59:19
tags: [Presto, Hive]
categories: OLAP
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Presto和Hive作为大数据分析领域OLAP工具，他们之间有什么区别和联系呢？下面来看看。

<img src="Presto和Hive的对比/view.jpeg" width="500" height="300"/>

# 1.本质区别
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Hive是把一个查询转化成多个MapReduce任务，然后一个接一个执行。执行的中间结果通过对磁盘的读写来同步。然而，Presto没有使用MapReduce，它是通过一个定制的查询和执行引擎来完成的。它的所有的查询处理是在内存中，这也是它的性能很高的一个主要原因。

# 2.执行速度
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;presto由于是基于内存的，而hive是在磁盘上读写的，因此presto比hive快很多，但是由于是基于内存的当多张大表关联操作时易引起内存溢出错误

# 3.处理json类型的数据
presto处理如下：
```
select 
    json_extract_scalar(xx['custom'],'$.position')
from table
```

hive处理如下：
```
select 
    get_json_object(xx['custom'],'$.position')
from table
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此外Presto还有一个函数json_extract是直接返回一个json串，根据需要自己需要选择函数

# 4.列转行
Hive:
```
select student, score from tests lateral view explode(split(scores, ',')) t as score;
```

Presto:
```
select student, score from tests cross json unnest(split(scores, ',') as t (score)
```


- - -
<b>Where there is a will, there is a way.</b>