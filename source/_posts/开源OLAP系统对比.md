---
title: 开源OLAP系统对比
date: 2019-10-09 16:06:49
tags: [OLAP,数据仓库]
categories: 数据仓库
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OLAP系统，基于数据仓库，为数据价值而生，提供复杂的数据分析操作，及直观易懂的查询结果，目前市场上有许多开源的OLAP框架，如Impala、Presto、Druid、Kylin、Spark SQL等，他们的关系又是怎样的呢，我们应该怎样选择框架？

![olap_analysis](开源OLAP系统对比/olap_analysis.jpeg)

<!-- more -->

# 1.Presto
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;facebook开源的一个java写的分布式数据查询框架，原生集成了hive、hbase和关系型数据库，Presto背后所使用的执行模式与hive有本质的不同，它没有使用MapReduce，比hive快一个数据量级，其中的关键是所有的计算都在内存中完成。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;缺点就是容易造成oom。

# 2.Druid
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Druid是一个实时处理时序数据的OLAP系统，采用预计算的方式，因为它的索引首先是按照时间的方式分片，查询的时候也是按照时间片去路由索引的。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;缺点就是sql支持不太好，需要使用它自己的方言来实现。

# 3.Spark SQL
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基于spark平台的一个OLAP框架，本质上也是基于DAG的MPP方案，基本思路是增加机器来提高并行计算能力，从而提高查询响应速度。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;缺点就是只能使用sql去即时查询，一次查询结果不能复用。

# 4.Kylin
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ebay开源框架，支持大数据生态圈的数据分析业务，核心是Cube，cube是一种预计算技术，主要是通过预计算的方式将用户预定义的多维立方体cube缓存起来，达到提高查询速度的目的，应用场景是针对复杂的sql join操作情况。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用空间来换时间，存储空间一般是数据仓库的10倍+。

# 5.Impala
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cloudera公司开源，能查询HDFS和Hbase中的数据，同Hive一样，也是一种SQL on Hadoop解决方案，它抛弃了MapReduce，而是使用传统的MPP数据库方式来提高查询速度。

- - -
You never know what's coming for you, so do it and don't care about the result.