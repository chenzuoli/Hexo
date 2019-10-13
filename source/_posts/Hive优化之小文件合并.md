---
title: Hive优化之小文件合并
date: 2019-10-13 15:25:01
tags: [Hive,优化]
categories: Hive
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当Hive输入由很多个小文件组成，由于每个小文件都会启动一个map任务，如果文件过小，以至于map任务启动和初始化的时间大于逻辑处理的时间，会造成资源浪费，甚至OOM。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为此，当我们启动一个任务，发现输入数据量小但任务数量多时，需要注意在Map前端进行输入合并。当然，在我们向一个表写数据时，也需要注意输出文件大小。

<img src="Hive优化之小文件合并/small_file_combine.jpeg" width="500" height="300"/>

<!-- more -->
# 问题
## 1.通常情况下，作业会通过input的目录产生一个或者多个map任务。 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主要的决定因素有： input的文件总个数，input的文件大小，集群设置的文件块大小(目前为128M, 可在hive中通过set dfs.block.size;命令查看到，该参数不能自定义修改)；

## 2.举例： 
a) 假设input目录下有1个文件a,大小为780M,那么hadoop会将该文件a分隔成7个块（6个128m的块和1个12m的块），从而产生7个map数
b) 假设input目录下有3个文件a,b,c,大小分别为10m，20m，130m，那么hadoop会分隔成4个块（10m,20m,128m,2m）,从而产生4个map数,即如果文件大于块大小(128m),那么会拆分，如果小于块大小，则把该文件当成一个块。

## 3.是不是map数越多越好？ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;答案是否定的。如果一个任务有很多小文件（远远小于块大小128m）,则每个小文件也会被当做一个块，用一个map任务来完成，而一个map任务启动和初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费。而且，同时可执行的map数是受限的。

## 4.是不是保证每个map处理接近128m的文件块，就高枕无忧了？ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;答案也是不一定。比如有一个127m的文件，正常会用一个map去完成，但这个文件只有一个或者两个小字段，却有几千万的记录，如果map处理的逻辑比较复杂，用一个map任务去做，肯定也比较耗时。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;针对上面的问题3和4，我们需要采取两种方式来解决：即减少map数和增加map数；

## 5.如何合并小文件，减少map数？ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;假设一个SQL任务：
```
Select count(1) from popt_tbaccountcopy_mes where pt = ‘2012-07-04’;
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该任务的inputdir  `/group/p_sdo_data/p_sdo_data_etl/pt/popt_tbaccountcopy_mes/pt=2012-07-04` 共有194个文件，其中很多是远远小于128m的小文件，总大小9G，正常执行会用194个map任务。Map总共消耗的计算资源： `SLOTS_MILLIS_MAPS= 623,020`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我通过以下方法来在map执行前合并小文件，减少map数：
```
set mapred.max.split.size=100000000;
set mapred.min.split.size.per.node=100000000;
set mapred.min.split.size.per.rack=100000000;
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;再执行上面的语句，用了74个map任务，map消耗的计算资源：`SLOTS_MILLIS_MAPS= 333,500`, 对于这个简单SQL任务，执行时间上可能差不多，但节省了一半的计算资源。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;大概解释一下，100000000表示100M, `set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;`这个参数表示执行前进行小文件合并，前面三个参数确定合并文件块的大小，大于文件块大小128m的，按照128m来分隔，小于128m,大于100m的，按照100m来分隔，把那些小于100m的（包括小文件和分隔大文件剩下的），进行合并,最终生成了74个块。
        
## 6.如何适当的增加map数？ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当input的文件都很大，任务逻辑复杂，map执行非常慢的时候，可以考虑增加Map数，来使得每个map处理的数据量减少，从而提高任务的执行效率。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;假设有这样一个任务：
```
Select 
    data_desc,
    count(1),
    count(distinct id),
    sum(case when …),
    sum(case when ...),
    sum(…)
from a group by data_desc;
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果表a只有一个文件，大小为120M，但包含几千万的记录，如果用1个map去完成这个任务，肯定是比较耗时的，这种情况下，我们要考虑将这一个文件合理的拆分成多个，这样就可以用多个map任务去完成:
```
set mapred.reduce.tasks=10;
create table a_1 as 
select 
    * 
from a 
distribute by rand(123); 
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这样会将a表的记录，随机的分散到包含10个文件的a_1表中，再用a_1代替上面sql中的a表，则会用10个map任务去完成。每个map任务处理大于12M（几百万记录）的数据，效率肯定会好很多。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;看上去，貌似这两种有些矛盾，一个是要合并小文件，一个是要把大文件拆成小文件，这点正是重点需要关注的地方，根据实际情况，控制map数量需要遵循两个原则：使大数据量利用合适的map数；使单个map任务处理合适的数据量；

# 1. Map输入合并小文件
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对应参数：
```
set mapred.max.split.size=256000000;            #每个Map最大输入大小
set mapred.min.split.size.per.node=100000000;   #一个节点上split的至少的大小 
set mapred.min.split.size.per.rack=100000000;   #一个交换机下split的至少的大小
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;  #执行Map前进行小文件合并
```

在开启了org.apache.hadoop.hive.ql.io.CombineHiveInputFormat后，一个data node节点上多个小文件会进行合并，合并文件数由
```
mapred.max.split.size           #限制的大小决定
mapred.min.split.size.per.node  #决定了多个data node上的文件是否需要合并~
mapred.min.split.size.per.rack  #决定了多个交换机上的文件是否需要合并~
```

# 2. 输出合并
```
set hive.merge.mapfiles=true;                 #在Map-only的任务结束时合并小文件
set hive.merge.mapredfiles=true;              #在Map-Reduce的任务结束时合并小文件
set hive.merge.size.per.task=256*1000*1000;   #合并文件的大小
set hive.merge.smallfiles.avgsize=16000000;   #当输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件merge
```

- - -
