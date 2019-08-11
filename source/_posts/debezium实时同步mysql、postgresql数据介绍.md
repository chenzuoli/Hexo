---
title: debezium实时同步mysql、postgresql数据介绍
date: 2019-07-20 11:01:09
tags: [实时同步,mysql,oracle,postgresql,mongo,sql server]
categories: 组件
---
给大家介绍一个实时同步数据库的组件debezium，它可以同步mysql、postgresql、mongo、oracle、sql server数据库到hdfs、kafka，功能强大，具体如下。
<!-- more -->

大致安装步骤：

1.插件decoderbufs或者wal2json
2.postgresql配置logical等
3.confluent平台搭建
4.配置connector

参考文档：<https://debezium.io/docs/connectors/>


<h1>for postgresql to kafka</h1>

<b>核心：一个active slot，多个connector</b>

## 1. bin/connect-distributed etc/kafka/connect-distributed.properties
三台服务器分别执行启动distributed服务（相同的slot.name、group.id保证复制同一个slot replication，保证在同一个组内）
```
 bin/connect-distributed etc/kafka/connect-distributed-8084.properties
 bin/connect-distributed etc/kafka/connect-distributed-8085.properties
 bin/connect-distributed etc/kafka/connect-distributed-8086.properties
```

## 2. bin/connect-standalone etc/kafka/connect-standalone.properties etc/kafka-connect-postgres/debezium.properties
bin/kafka-console-consumer --zookeeper localhost:2182 --topic postgres.localhost.public.test --from-beginning

## 3. 安装decoderbufs、wal2json plugin

<https://github.com/debezium/postgres-decoderbufs/blob/master/README.md>
<https://github.com/eulerto/wal2json/blob/master/README.md>

安装wal2json时出现的问题：Makefile:10: /usr/lib64/pgsql/pgxs/src/makefiles/pgxs.mk: No such file or directory
yum install postgresql10-devel即可

## 4. 配置

参考文档：<https://zhubingxu.me/2018/06/05/debezium-postgres/>

share/java/下创建debezium文件夹，创建文件debezium.properties：
```
name=events-debezium
tasks.max=1
connector.class=io.debezium.connector.postgresql.PostgresConnector
database.hostname=localhost
database.port=5432
database.user=postgres
database.password=postgres
database.dbname=postgres
database.history.kafka.bootstrap.servers=localhost:9092
database.server.id=1
database.server.name=postgres.localhost
plugin.name=wal2json
include.schema.changes=true
slot.name=my\_slot\_name
```

## 5. 异常

Error while fetching metadata with correlation id 1 : {postgres.localhost.public.test=LEADER\_NOT\_AVAILABLE}

<https://stackoverflow.com/questions/35788697/leader-not-available-kafka-in-console-producer>

配置debezium中$DEBEZIUM\_HOME/etc/kafka/server.properties

指定参数advertised.host.name

ERROR A logical replication slot named 'debezium' for plugin 'wal2json' and database 'postgres' is already active on the server.You cannot have multiple slots with the same name active for the same database (io.debezium.connector.postgresql.connection.PostgresReplicationConnection:104)

在创建connector的配置参数中添加新的slot.name，slot.name的规范必须为字母数字下划线。不指定的话默认为debezium，会产生冲突。

## 6. 分布式kafka connector配置：

<https://archive.cloudera.com/kafka/kafka/2/kafka-0.9.0-kafka2.0.1/connect.html>分布式connector需要通过rest api进行增删改

<https://mapr.com/docs/52/Kafka/Connect-distributed-mode.html> connect-distributed.properties

连接参数配置：[https://debezium.io/docs/connectors/postgresql/\#connector-properties](https://debezium.io/docs/connectors/postgresql/#connector-properties)

## 7. 注意：

table.whitelist格式：schemaName.tblname

如果启动connector出现权限不足时：需要给用户赋update权限：

GRANT SELECT, UPDATE ON TABLE test TO debezium;

不监控无主键的表

## 8. 查看postgresql slot

a.登陆：psql -U user -d db -h host -p port -W
b.查看所有slot： select \* from pg\_replication\_slots;
c.添加slot：SELECT \* FROM pg\_create\_physical\_replication\_slot('pg96\_102');
d.删除slot：SELECT \* FROM pg\_drop\_replication\_slot('pg96\_102');



</br>
</br>

<h1>for mysql to kafka</h1>

核心：启动多个connector即可

## 1. 账户及权限

mysql user: debezium:\*\*\*\*\*\*\*

GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON \*.\* TO 'debezium' IDENTIFIED BY '\*\*\*\*\*\*';

## 2. 配置

[https://debezium.io/docs/connectors/mysql/\#setting-up-mysql](https://debezium.io/docs/connectors/mysql/#setting-up-mysql)

## 3. 启动

bin/connect-standalone etc/kafka/connect-standalone.properties share/java/debezium/mysql-debezium-8087.properties

三台服务器分别运行：
```
bin/connect-distributed etc/kafka/mysql-connect-distributed-8134.properties
bin/connect-distributed etc/kafka/mysql-connect-distributed-8134.properties
bin/connect-distributed etc/kafka/mysql-connect-distributed-8134.properties
```

## 4. kafka connector

请求添加connector-task，随便在哪一台服务器添加1个task

curl -X POST -H "Content-Type: application/json" --data @share/java/debezium/mysql-8134.json <http://localhost:8134/connectors>

table.whitelist格式：dbname.tblname

测试环境test-hadoop：/mnt/confluent下
生成的topic名称为：servername.dbname.tblname

## 5. 异常（未解决）

![1](debezium实时同步mysql、postgresql数据介绍/5DCA7786FF43D058BBB827BC521EFBDE)
![2](debezium实时同步mysql、postgresql数据介绍/A91E335CC16D805E66B44CD76ADF4383)

<b>如果有解决方案，请联系我邮箱chenzuoli709@163.com，不胜感激。</b>