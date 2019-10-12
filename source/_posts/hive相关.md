---
title: hive相关
date: 2019-07-21 23:01:10
tags: hive
categories: Hive
---

下面的内容包含hive的简单操作，增删改查，权限，一些异常的解决方案。

<!-- more -->

# 1. 在linux命令行端执行hql语句
```
hive -e 'show table tableName'
```
# 2. 在linux命令行端执行hql文件
```
hive -f fileName
```
# 3. 按照分区查看hive表中的数据量
```
hive -e 'select dt, count(1) from test_hive.wps_android_uuid_userid group by dt'
```
# 4. 添加分区
```
alter table test_hive.wps_android_uuid_userid  add  if not exists partition (dt='2018-08-04')
```
# 5. 赋表权限
```
grant select on table usersdb.account_src to user w_wangzhe
```
# 6. 赋库权限
```
GRANT ALL ON DATABASE DEFAULT TO USER fatkun;
GRANT ALL ON TABLE test TO GROUP kpi;
REVOKE ALL ON TABLE test FROM GROUP kpi;
GRANT ALL TO USER fatkun;
REVOKE ALL FROM fatkun;
```
# 7. 重命名
```
alter table data_platform.td_request_log rename to data_platform.td_request_log_old
```
# 8. Error 
while processing statement: FAILED: Execution Error, return code 2 from org.apache.hadoop.hive.ql.exec.mr.MapredLocalTask
```
set hive.auto.convert.join = false;
```
        
# 9.执行sql文件
```
hive -d etldate='2018-08-16' -f loan_periods.hql
```
# 10.add jar
```
add jar /usr/hdp/current/hive-client/lib/commons-httpclient-3.0.1.jar;
```
# 11.add column
```
hive -e "alter table report.user_detail_20180614  add columns(identifier_type string comment '注册类型',channel string comment '注册渠道')"
```
# 12.load 文件到hive表
本地文件：
```
hive -e "load data local inpath '/data/code/app_list_0814.csv' into table dim.dim_app_list"
```
hdfs文件：
```
hive -e "load data inpath '/data/code/app_list_0814.csv' into table dim.dim_app_list"
```
# 13.列
## 13.1修改列位置
alter table factor.mf_bus_finc_app change column submit_op_no submit_op_no string after company_id
## 13.2增加列
hive -e "alter table data_platform_new.face_request_log add columns(channel string)"
# 14.修改权限
hive -e "grant select on table stg.risk_apply_users to user userName"
# 15.自定义函数
```
create  function dateformat as 'com.kso.dw.hive.udf.DateFormat' using jar 'hdfs://hdfs-ha/hiveudf/dw_hive_udf.jar';
mysql -h10.0.1.160 -uadmin -pmd854NHmv3bF0kl9 hive4fac31f3 -e 'select name from dbs' | xargs -n 1 -i echo "create  function  {}.mymd5_kc as 'com.kso.dw.hive.udf.MyMd5_KeyCenter' using jar 'hdfs://hdfs-ha/hiveudf/dw_hive_udf.jar';"
```
# 16.全局替换
```
sed -i 's/CREATE TABLE/CREATE EXTERNAL TABLE/g' *.hql
```
# 17.not a file exception
not a file ks3://online-hadoop/ods/report/dt=2019-01-01/1
```
set mapreduce.input.fileinputformat.input.dir.recursive=true;
```
# 18.exception
```
set hive.execution.engine=mr;
```

# 19. too many counters
org.apache.hadoop.mapreduce.counters.LimitExceededException: Too many counters: 121 max=120
resolved:change the tez configuration
```
tez.counters.max= 200
```

# 20.SHOW TRANSACTIONS
```
ABORT TRANSACTIONS 4951;
show locks;
mysql:
select * from hive_locks;
select * from hive_locks where HL_TXNID > 0;
```
# 21.acquiring locks
FAILED: Error in acquiring locks: Lock acquisition for LockRequest(component:[LockComponent(type:EXCLUSIVE
关闭事务： set hive.support.concurrency=false
# 22.事务表查询
```
set hive.support.concurrency=true;
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.txn.manager=org.apache.hadoop.hive.ql.lockmgr.DbTxnManager;
set hive.compactor.initiator.on=true;
set hive.compactor.worker.threads=1;
```
# 23. 内部表转外部表
```
alter table default.test set TBLPROPERTIES('EXTERNAL'='true');
```

# 24. 外部表转内部表
```
alter table tableA set TBLPROPERTIES('EXTERNAL'='false')
```

# 25.修改元数据路径
    元数据库：
```
UPDATE dbs SET DB_LOCATION_URI=REPLACE(DB_LOCATION_URI,'hdfs-ha','bjCluster');
``` 
    元数据表：
```
UPDATE sds SET LOCATION=REPLACE(LOCATION,'ks-jinrong-dw','online-hadoop');
UPDATE sds SET LOCATION=REPLACE(LOCATION,'hdfs-ha','bjCluster');
```
    自定义函数：
```
UPDATE func_ru SET RESOURCE_URI=REPLACE(RESOURCE_URI,'hdfs-ha','bjCluster');
```

# 26.set role admin
# 27.控制map个数
```
set mapred.max.split.size=400000000;
set mapred.min.split.size.per.node=400000000;
set mapred.min.split.size.per.rack=400000000;
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
```
