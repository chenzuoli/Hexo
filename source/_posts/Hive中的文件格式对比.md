---
title: Hive中的文件格式对比
date: 2019-10-12 10:59:43
tags: [Hive,文件格式]
categories: Hive
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Hive表数据实际存储在HDFS文件系统中，而不同的文件格式，会有不同的特性，我们在数据仓库建设中，如何根据仓库不同层次特点设计不同的文件格式呢？下面来看下。

<!-- more -->

# 一、hive中的file_format
- SEQUENCEFILE：行存储，生产中绝对不会用，k-v格式，二进制文件，比源文本格式占用磁盘更多；
- TEXTFILE：行存储，生产中用的多，行式存储。最简单的数据格式，便于和其他工具（Pig, grep, sed, awk）共享数据，便于查看和编辑；
- RCFILE：行列混合存储，生产中用的少；
- ORC：列式存储，生产中最常用，OCFILE的升级版，查询效率最高。
- PARQUET：列式存储，生产中最常用，更高效的压缩和编码；
- AVRO：生产中几乎不用，不用考虑
- JSONFILE：生产中几乎不用，不用考虑
- INPUTFORMAT：生产中几乎不用，不用考虑
注意hive默认的文件格式是TextFile，可通过 `set hive.default.fileformat` 进行配置

![1](Hive中的文件格式对比/1.png)

# 二、行式存储与列式存储
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;行式存储与列式存储数据物理底层存储区别

![2](Hive中的文件格式对比/2.png)

# 结论：由上图可知
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`行式存储`一定会把同一行数据存到同一个块中，在`select`查询的时候，是对所有字段的查询，不可以单独查询某一行
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`列式存储`同一列数据一定是存储到同一个块中，换句话说就是不同的列可以放到不同块中，在进行select查询的时候可以单独查询某一列。

# 三、优缺点
## 1.列式存储
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>优点</b>：当查询某个或者某几个字段的时候，只需要查看存储这几个字段的这几个`block`就可以了，大大的减少了数据的查询范围，提高了查询效率
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>缺点</b>：当进行全字段查询的时候，数据需要重组，比单独查一行要慢

## 2.行式存储
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>优点</b>：全字段查询比较快
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>缺点</b>：当查询一张表里的几个字段的时候，底层依旧是读取所有的字段，这样查询效率降低，并且会造成不必要的资源浪费，而且，生产中很少会出现需要全字段查询的场景

# 四、hive文件格式配置实现以及对比
## 1.创建原始表默认TEXTFILE
```
CREATE EXTERNAL TABLE g6_access (
    cdn string,
    region string,
    level string,
    time string,
    ip string,
    domain string,
    url string,
    traffic bigint
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
LOCATION '/g6/hadoop/access/clear/test/';
```

导入测试数据
```
[hadoop@hadoop001 data]$ ll
-rw-r--r-- 1 hadoop hadoop 68051224 Apr 17 17:37 part-r-00000
[hadoop@hadoop001 data]$ hadoop fs -put part-r-00000 /g6/hadoop/access/clear/test/
```
通过hue查看数据的大小    64.9MB

![3](Hive中的文件格式对比/3.png)

## 2.创建以 SEQUENCEFILE格式储存的表g6_access_seq，并使用g6_access中的数据
```
create table g6_access_seq
stored as SEQUENCEFILE
as select * from g6_access;
```
查看数据大小   71.8MB

![4](Hive中的文件格式对比/4.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结论：比默认的TEXTFILE格式的文件还要大，生产上基本上是不会用的

## 3.创建RCFILE数据存储格式表，，并使用g6_access中的数据
```
create table g6_access_rc
stored as RCFILE
as select * from g6_access;
```
查看数据大小  61.6MB

![5](Hive中的文件格式对比/5.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结论：存储减少了3M左右，微不足道，读写性能也没有什么优势，生产也没有用他的理由

## 4.创建ORCFILE数据存储格式表，，并使用g6_access中的数据，默认是使用zlib压缩，支持`zlib`和`snappy`
```
create table g6_access_orc
stored as ORC
as select * from g6_access;
``` 

查看数据大小    17.0MB

![6](Hive中的文件格式对比/6.png)

## 5.创建ORCFILE数据存储格式表，不使用压缩，并使用g6_access中的数据
```
create table g6_access_orc_none
stored as ORC tblproperties ("orc.compress"="NONE")
as select * from g6_access;
```
查看数据大小    51.5MB

![7](Hive中的文件格式对比/7.png)

## 6.创建PARQUET数据存储格式表，不使用压缩，并使用g6_access中的数据
```
create table g6_access_par
stored as PARQUET
as select * from g6_access;
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结论：ORC文件不压缩，比源文件少了10多MB，ORC文件采用默认压缩，文件只有源文件的四分之一
查看数据大小    58.3MB

## 7.创建PARQUET数据存储格式表，设置使用gzip压缩格式，并使用g6_access中的数据
```
set parquet.compression=gzip;
create table g6_access_par_zip
stored as PARQUET
as select * from g6_access;
```

![8](Hive中的文件格式对比/8.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结论：parquet格式文件大小是源文件的1/4左右。生产上也是好的选择

# 五、读取数据量对比
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;直接执行 select 查询，观察日志 尾部HDFS Read: 190XXX ,就可知道读取数据量了。

# 六、总结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>行存储</b>：查询满足条件的一整行数据的时候，列存储则需要去每个聚集的字段找到对应的每个列的值，行存储只需要找到其中一个值，其余的值都在相邻地方，所以此时行存储查询的速度更快。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>列存储</b>：因为每个字段的数据聚集存储，在查询只需要少数几个字段的时候，能大大减少读取的数据量；每个字段的数据类型一定是相同的，列式存储可以针对性的设计更好的设计压缩算法。