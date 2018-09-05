---
title: 阿里巴巴mysql数据库binlog的增量订阅与消费组件canal
date: 2018-09-05 20:15:44
tags: [canal,监控,mysql]
categories: 监控组件
---
首先介绍下canal他可以做什么，基于日志增量订阅&消费支持的业务, 监控mysql数据，将mysql增量数据从binlog中获取过来实现数据库的镜像、数据库实时备份、多级索引、业务cache刷新等，具体参考阿里开源项目代码：
<a href="https://github.com/alibaba/canal">canal github</a>
<!-- more -->
# canaldbkafka
## 简介
canaldbkafka是连接canal和kafka的一个中间件。目的是实现数据库某个表格数据变更转变成消息流的形式，以便后续业务消费kafka的消息流。 canal wiki:https://github.com/alibaba/canal/wiki
## 消息的类型
canal的binlog 会被解析成以下3中类型的消息。其他的类型被过滤掉了。
### insert

```
{
    "data": {
        "need_sub": {
            "type": "int(11)",
            "updated": true,
            "value": "0"
        },
        "order_description": {
            "type": "varchar(1024)",
            "updated": true,
            "value": ""
        },
        "pay_amount": {
            "type": "int(11)",
            "updated": true,
            "value": "0"
        },
        "pay_order": {
            "type": "varchar(30)",
            "updated": true,
            "value": ""
        }
    },
    "type": "insert"
}
```

### delete
```
{
    "data": {
        "need_sub": {
            "type": "int(11)",
            "updated": true,
            "value": "0"
        },
        "order_description": {
            "type": "varchar(1024)",
            "updated": true,
            "value": ""
        },
        "pay_amount": {
            "type": "int(11)",
            "updated": true,
            "value": "0"
        },
        "pay_order": {
            "type": "varchar(30)",
            "updated": true,
            "value": ""
        }
    },
    "type": "delete"
}
```
### update
data对象是各字段类型、是否被更新、值。olddata对象是之前的状态。

```
{
    "data": {
        "Quota": {
            "type": "tinyint(4)",
            "updated": false,
            "value": "0"
        },
        "ReqAmount": {
            "type": "int(11)",
            "updated": true,
            "value": "100"
        }
    },
    "olddata": {
        "Quota": {
            "type": "tinyint(4)",
            "updated": false,
            "value": "0"
        },
        "ReqAmount": {
            "type": "int(11)",
            "updated": false,
            "value": "0"
        }
    },
    "type": "update"
}
```

## 使用说明
### 编译安装

```
mvn compile

mvn package

ll target/canal-dbkafka   #可部署
total 0
drwxr-xr-x   5 xxx  staff   170B 12 21 21:26 bin
drwxr-xr-x   3 xxx  staff   102B 12 21 21:26 conf
drwxr-xr-x  24 xxx  staff   816B 12 21 21:26 lib
drwxr-xr-x   2 xxx  staff    68B 12 21 21:26 logs

ll target/canal-dbkafka/bin  #startmy.sh为启动示例
-rwxr-xr-x  1 xxx  staff   271B 12 21 21:26 startmy.sh
-rwxr-xr-x  1 xxx  staff   2.5K 12 21 21:26 startup.sh
-rwxr-xr-x  1 xxx  staff   1.0K 12 21 21:26 stop.sh

```
### 启动说明
已startmy.sh为例
```
#!/bin/bash

current_path=`pwd`
case "`uname`" in
    Linux)
        bin_abs_path=$(readlink -f $(dirname $0))
        ;;
    *)
        bin_abs_path=`cd $(dirname $0); pwd`
        ;;
esac
cd ${bin_abs_path} && ./startup.sh testdb thetable 127.0.0.1:2181 127.0.0.1:9092
```
1. testdb 是canal配置的destination
2. thetable kafka的具体topic
3. 127.0.0.1:2181 是canal配置HA 对应的zookeeper的地址
4. 127.0.0.1:9092  是kafka的地址


### 使用注意事项
1. mysql binlog模式设置为row模式
2. 为了保证数据库消息的顺序性，将消息存储kafka的时候组件采用了同步的方式
3. canal 必须配置zookeeper ha的模式 https://github.com/alibaba/canal/wiki/AdminGuide#ha%E6%A8%A1%E5%BC%8F%E9%85%8D%E7%BD%AE
4. 之前使用针对的是数据库中的一个表在canal配置中已经过滤所以消息中没有表名 可以说是个设计的缺陷。

# 高可用及分布式
## 监控多个mysql
canal分服务端和客户端，我们需要监控多个mysql时，可以配置多个instance，具体编辑服务端配置文件canal.properties：
```
#################################################
############     destinations       #############
#################################################
canal.destinations=dest21,dest14
# conf root dir
canal.conf.dir = ../conf
# auto scan instance dir add/remove and start/stop instance
canal.auto.scan = true
canal.auto.scan.interval = 5
```
其中dest21和dest14为不同的instance，目录结构如下：
```
-rwxr-xr-x 1 root root 2882 Aug 27 18:44 canal.properties
drwxr-xr-x 2 root root 4096 Sep  5 19:08 dest14
drwxr-xr-x 2 root root 4096 Sep  5 19:09 dest21
-rwxr-xr-x 1 root root 3038 Jun 19 17:18 logback.xml
drwxr-xr-x 3 root root 4096 Jun 19 17:18 spring
```
dest14和dest21目录分别为监控不同mysql的配置文件放置位置，具体如下：
```
#################################################
## mysql serverId
canal.instance.mysql.slaveId=14

# position info
canal.instance.master.address=1.1.1.1:3306
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=


# table meta tsdb info
canal.instance.tsdb.enable=true
canal.instance.tsdb.dir=${canal.file.data.dir:../conf}/${canal.instance.destination:}
canal.instance.tsdb.url=jdbc:h2:${canal.instance.tsdb.dir}/h2;CACHE_SIZE=1000;MODE=MYSQL;
#canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
canal.instance.tsdb.dbUsername=canal
canal.instance.tsdb.dbPassword=canal


#canal.instance.standby.address =
#canal.instance.standby.journal.name =
#canal.instance.standby.position =
#canal.instance.standby.timestamp =

# username/password
canal.instance.dbUsername=canal
canal.instance.dbPassword=*****
canal.instance.defaultDatabaseName=
canal.instance.connectionCharset=UTF-8

# table regex
#canal.instance.filter.regex=.*\\..*
canal.instance.filter.regex=event_collection\.user_location_lng_lat
# table black regex
canal.instance.filter.black.regex=
#################################################
```
你需要修改的地方：
```
canal.instance.mysql.slaveId -- 不同的instance分配不同的slaveId，因为canal监控mysql的原理就是伪装成mysql的slave来获取binlog日志的
canal.instance.master.address -- 配置监控的mysql ip地址
canal.instance.dbUsername -- 连接mysql的用户名
canal.instance.dbPassword -- 连接mysql的密码
canal.instance.filter.regex -- 监控mysql中的哪个库，哪个表
```
其中监控mysql的哪个库哪个表编写格式如下：
```
.*\\..*  --表示监控mysql所有库所有表
test\..*  --表示监控mysql test库下的所有表
test\.test  --表示监控mysql test库下的test表
```

阿里巴巴，我们程序员的梦想，开源的canal还是不错的，希望大家借助这篇文章能够熟练掌握canal的简单使用，如果遇到什么问题，欢迎一起讨论，在下方留言或者mail我：chenzuoli@gmail.com