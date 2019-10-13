---
title: Hive文件压缩格式对比
date: 2019-10-13 11:02:11
tags: [Hive,Bzip2,Gzip,Snappy,Lzo,压缩格式]
categories: Hive
notebook: Hive
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Hive的文件压缩格式应该在不同的应用场景下使用不同的方式，例如cpu资源足够，但是硬盘容量不足时，可以使用bzip2方式。也跟文件格式有关，对列式存储的文件进行压缩，会得到一个可观的压缩比例，我们在上一篇文章中讲解了 <a>[<font color=#0099ff>Hive文件格式的对比</font>](http://wetech.top/2019/10/12/Hive%E4%B8%AD%E7%9A%84%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E5%AF%B9%E6%AF%94/)。

![zipper](Hive文件压缩格式对比/zip.jpeg)

<!-- more -->
# 一、对比
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们来看下不同的压缩方式的对比：
```
/*
压缩：可以最小化所需要的磁盘存储空间，以及减小磁盘和网络I/O操作，但是文件压缩和解压过程会增加CPU开销。因此，对于压缩密集型的job最好使用压缩，特别是有额外的CPU资源或者磁盘存储空间比较稀缺的情况。
*/
-- BZip2压缩率最高，但是消耗最多的CPU开销
-- GZip是压缩率和压缩/解压速度上的下一个选择
-- LZO和Snappy压缩率低于前两者，但是速度快
-- Bzip2和LZO可以支持块级别的压缩，另外两者不支持，如果用后两者，可以将文件分割成期望值的大小。
```

# 二、配置
## 1.配置文件（永久有效）
```
[hadoop@hadoop004 hadoop]$ vim core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop004:9000</value>
    </property>
    <property>
        <name>io.compression.codecs</name>
        <value>org.apache.hadoop.io.compress.GzipCodec,
                org.apache.hadoop.io.compress.DefaultCodec,
                org.apache.hadoop.io.compress.BZip2Codec,
                org.apache.hadoop.io.compress.SnappyCodec
        </value>
    </property>
</configuration>
```

```
[hadoop@hadoop004 hadoop]$ vim mapred-site.xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <!-- 配置 Map段输出的压缩,snappy-->
    <property>
        <name>mapreduce.map.output.compress</name>
        <value>true</value>
    </property>
    <property>
        <name>mapreduce.map.output.compress.codec</name>
        <value>org.apache.hadoop.io.compress.SnappyCodec</value>
    </property>
    
    <!--开启MapReduce输出文件压缩-->
    <property>
        <name>mapreduce.output.fileoutputformat.compress</name>
        <value>true</value>
    </property>
    <property>
        <name>mapreduce.output.fileoutputformat.compress.codec</name>
        <value>org.apache.hadoop.io.compress.BZip2Codec</value>
    </property>
</configuration>
```

### 1.1 先看看snappy的效果
```
[hadoop@hadoop004 data]$ hdfs dfs -du -s -h /user/hive/warehouse/page_views/page_views.dat
18.1 M  18.1 M  /user/hive/warehouse/page_views/page_views.dat
```
```
hive> SET hive.exec.compress.output;
hive.exec.compress.output=false
hive>
    > create table page_views(
    > track_times string,
    > url string,
    > session_id string,
    > referer string,
    > ip string,
    > end_user_id string,
    > city_id string
    > ) row format delimited fields terminated by '\t';
OK
Time taken: 0.741 seconds
 
hive> load data local inpath '/home/hadoop/data/page_views.dat' overwrite into table page_views;
Loading data to table default.page_views
Table default.page_views stats: [numFiles=1, numRows=0, totalSize=19014993, rawDataSize=0]
OK
Time taken: 0.595 seconds
```

因为我在mapred-site.xml里面设置默认的解压缩格式为snappy

```
hive>
    > set hive.exec.compress.output=true;
hive> set hive.exec.compress.output;
hive.exec.compress.output=true
hive> set mapreduce.output.fileoutputformat.compress.codec;
mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec
hive> create table page_views_snappy as select * from page_views;
Query ID = hadoop_20190419141818_561ede29-e964-4655-9e48-1e4f5d6eeb5c
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1555643336639_0002, Tracking URL = http://hadoop004:8088/proxy/application_1555643336639_0002/
Kill Command = /home/hadoop/app/hadoop-2.6.0-cdh5.7.0/bin/hadoop job  -kill job_1555643336639_0002
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
2019-04-19 14:30:50,994 Stage-1 map = 0%,  reduce = 0%
2019-04-19 14:30:57,450 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 2.7 sec
MapReduce Total cumulative CPU time: 2 seconds 700 msec
Ended Job = job_1555643336639_0002
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to: hdfs://hadoop004:9000/user/hive/warehouse/.hive-staging_hive_2019-04-19_14-30-44_413_8511625915872870114-1/-ext-10001
Moving data to: hdfs://hadoop004:9000/user/hive/warehouse/page_views_snappy
Table default.page_views_snappy stats: [numFiles=1, numRows=100000, totalSize=8814444, rawDataSize=18914993]
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1   Cumulative CPU: 2.7 sec   HDFS Read: 19018292 HDFS Write: 8814535 SUCCESS
Total MapReduce CPU Time Spent: 2 seconds 700 msec
OK
Time taken: 15.354 seconds
```
```
[hadoop@hadoop004 data]$ hdfs dfs -ls /user/hive/warehouse/page_views_snappy
Found 1 items
-rwxr-xr-x   1 hadoop supergroup    8814444 2019-04-19 14:30 /user/hive/warehouse/page_views_snappy/000000_0.snappy
 
[hadoop@hadoop004 data]$ hdfs dfs -du -s -h /user/hive/warehouse/page_views_snappy
8.4 M  8.4 M  /user/hive/warehouse/page_views_snappy
```

### 1.2 下面采用bzip2瞅瞅
```
hive>
    > set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.BZip2Codec;
 
hive> set mapreduce.output.fileoutputformat.compress.codec;
mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.BZip2Codec
```
```
hive> create table page_views_bzip2 as select * from page_views;
Query ID = hadoop_20190419143939_53d67293-92de-4697-a791-f9a1afe7be01
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1555643336639_0003, Tracking URL = http://hadoop004:8088/proxy/application_1555643336639_0003/
Kill Command = /home/hadoop/app/hadoop-2.6.0-cdh5.7.0/bin/hadoop job  -kill job_1555643336639_0003
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
2019-04-19 14:44:16,992 Stage-1 map = 0%,  reduce = 0%
2019-04-19 14:44:25,471 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 5.19 sec
MapReduce Total cumulative CPU time: 5 seconds 190 msec
Ended Job = job_1555643336639_0003
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to: hdfs://hadoop004:9000/user/hive/warehouse/.hive-staging_hive_2019-04-19_14-44-09_564_3458580979229190548-1/-ext-10001
Moving data to: hdfs://hadoop004:9000/user/hive/warehouse/page_views_bzip2
Table default.page_views_bzip2 stats: [numFiles=1, numRows=100000, totalSize=3815195, rawDataSize=18914993]
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1   Cumulative CPU: 5.19 sec   HDFS Read: 19018291 HDFS Write: 3815285 SUCCESS
Total MapReduce CPU Time Spent: 5 seconds 190 msec
OK
Time taken: 17.289 seconds
```
```
[hadoop@hadoop004 hadoop]$ hdfs dfs -ls /user/hive/warehouse/page_views_bzip2
Found 1 items
-rwxr-xr-x   1 hadoop supergroup    3815195 2019-04-19 14:44 /user/hive/warehouse/page_views_bzip2/000000_0.bz2
 
[hadoop@hadoop004 hadoop]$ hdfs dfs -du -s -h /user/hive/warehouse/page_views_bzip2/*
3.6 M  3.6 M  /user/hive/warehouse/page_views_bzip2/000000_0.bz2
```

### 1.3 再来看看gzip的
```
hive>
    > set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.GzipCodec;
 
hive> set mapreduce.output.fileoutputformat.compress.codec;
mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.GzipCodec
```
```
hive>
    > create table page_views_gzip as select * from page_views;
Query ID = hadoop_20190419143939_53d67293-92de-4697-a791-f9a1afe7be01
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1555643336639_0004, Tracking URL = http://hadoop004:8088/proxy/application_1555643336639_0004/
Kill Command = /home/hadoop/app/hadoop-2.6.0-cdh5.7.0/bin/hadoop job  -kill job_1555643336639_0004
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
2019-04-19 14:48:10,606 Stage-1 map = 0%,  reduce = 0%
2019-04-19 14:48:18,019 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 3.29 sec
MapReduce Total cumulative CPU time: 3 seconds 290 msec
Ended Job = job_1555643336639_0004
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to: hdfs://hadoop004:9000/user/hive/warehouse/.hive-staging_hive_2019-04-19_14-48-03_436_7531556390065383047-1/-ext-10001
Moving data to: hdfs://hadoop004:9000/user/hive/warehouse/page_views_gzip
Table default.page_views_gzip stats: [numFiles=1, numRows=100000, totalSize=5550655, rawDataSize=18914993]
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1   Cumulative CPU: 3.29 sec   HDFS Read: 19018290 HDFS Write: 5550744 SUCCESS
Total MapReduce CPU Time Spent: 3 seconds 290 msec
OK
Time taken: 15.866 seconds
```
```
[hadoop@hadoop004 hadoop]$ hdfs dfs -ls /user/hive/warehouse/page_views_gzip
Found 1 items
-rwxr-xr-x   1 hadoop supergroup    5550655 2019-04-19 14:48 /user/hive/warehouse/page_views_gzip/000000_0.gz
 
[hadoop@hadoop004 hadoop]$ hdfs dfs -du -s -h /user/hive/warehouse/page_views_gzip/*
5.3 M  5.3 M  /user/hive/warehouse/page_views_gzip/000000_0.gz
```

## 2.配置参数（当前session有效）
### 2.1. Hive中间数据压缩
  `hive.exec.compress.intermediate`：默认该值为false，设置为true为激活中间数据压缩功能。HiveQL语句最终会被编译成Hadoop的Mapreduce job，开启Hive的中间数据压缩功能，就是在MapReduce的shuffle阶段对mapper产生的中间结果数据压缩。在这个阶段，优先选择一个低CPU开销的算法。 
  `mapred.map.output.compression.codec`：该参数是具体的压缩算法的配置参数，SnappyCodec比较适合在这种场景中编解码器，该算法会带来很好的压缩性能和较低的CPU开销。设置如下：
```
set hive.exec.compress.intermediate=true;
set mapred.map.output.compression.codec= org.apache.hadoop.io.compress.SnappyCodec;
set mapred.map.output.compression.codec=com.hadoop.compression.lzo.LzoCodec;
```
### 2.2. Hive最终数据压缩
  `hive.exec.compress.output`：用户可以对最终生成的Hive表的数据通常也需要压缩。该参数控制这一功能的激活与禁用，设置为true来声明将结果文件进行压缩。 
  `mapred.output.compression.codec`：将`hive.exec.compress.output`参数设置成true后，然后选择一个合适的编解码器，如选择SnappyCodec。设置如下：
```
set hive.exec.compress.output=true;
set mapred.output.compression.codec=org.apache.hadoop.io.compress.SnappyCodec;
```

### 2.3. 建表：
Text:
```
${建表语句}
stored as textfile;
##########################################插入数据########################################
set hive.exec.compress.output=true; --启用压缩格式
set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;  --指定输出的压缩格式为Gzip 
set mapred.output.compress=true;   
set io.compression.codecs=org.apache.hadoop.io.compress.GzipCodec;     
insert overwrite table textfile_table select * from T_Name;
```

Sequence
```
${建表语句}
SORTED AS SEQUENCEFILE;    --将Hive表存储定义成SEQUENCEFILE
##########################################插入数据########################################
set hive.exec.compress.output=true; --启用压缩格式
set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec; --指定输出的压缩格式为Gzip 
set mapred.output.compression.type=BLOCK;   --压缩选项设置为BLOCK
set mapred.output.compress=true; 
set io.compression.codecs=org.apache.hadoop.io.compress.GzipCodec; 
insert overwrite table textfile_table select * from T_Name;
```

RCFile
```

${建表语句}
stored as rcfile;
-插入数据操作：
set hive.exec.compress.output=true; 
set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec; 
set mapred.output.compress=true; 
set io.compression.codecs=org.apache.hadoop.io.compress.GzipCodec; 
insert overwrite table rcfile_table select * from T_Name;
```

Parquet
```
set parquet.compression=snappy;
```
```
create table mytable(a int,b int) stored as parquet tblproperties('parquet.compression'='snappy');
```
```
alter table mytable set tblproperties ('parquet.compression'='snappy');
```

- - -
Trobule is Friend.