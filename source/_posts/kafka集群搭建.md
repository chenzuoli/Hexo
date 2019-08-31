---
title: kafka集群搭建
date: 2019-08-31 10:57:17
tags: kafka
categories: 环境搭建
---

kafka，领英开源消息队列框架，在大数据实时、批处理过程中，充当缓冲、中间存储、解耦的组件，深得工程师的喜爱，下面看下如何搭建。
![kafka-logo](kafka集群搭建/kafka-logo.png)
<!-- more -->

# 1、软件环境
搭建好的zookeeper集群
下载kafka安装包
# 2、安装、修改配置文件
```
tar -zxvf kafka_*.tgz -C /usr/local/
cd /usr/local/kafka/config
```
## 2.1修改配置文件
```
vim server.properties
broker.id=1
/* 这是这台虚拟机上的值，在另外两台虚拟机上应该是2或者3，这个值是唯一的，每台虚拟机或者叫服务器不能相同。 /
/ 设置本机IP和端口。 我这里设置的是listeners，也可以直接设置host.name=192.168.172.10,port=9092,这个IP地址也是与本机相关的，每台服务器上设置为自己的IP地址。 /
listeners=PLAINTEXT://192.168.172.10:9092
advertised.listeners=PLAINTEXT://x.x.x.x:9092 //外部访问
// 该端口默认是9092
// 在og.retention.hours=168下面新增下面三项 #默认消息的最大持久化时间，168小时，7天
message.max.byte=5242880    #消息保存的最大值5M
    default.replication.factor=2 #kafka保存消息的副本数，如果一个副本失效了，另一个还可以继续提供服务
replica.fetch.max.bytes=5242880 #取消息的最大直接数
/ 设置zookeeper的连接端口，新版本的kafka不再使用zookeeper而是通过brokerlist的配置让producer直接连接broker，这个brokerlist可以配置多个，只要有一个能连接上，就可以让producer获取道集群中的其他broker的信息，绕过了zookeeper。因此这个zookeeper.connect可以设置多个值 */
zookeeper.connect=192.168.172.12:2181,192.168.172.11:2181,192.168.172.10:2181
```
# 3、启动kafka
首先要启动kafka集群，并且是三台都要手动去启动。
// 进入kafka的bin目录
cd /opt/kafka/kafka_2.11-1.0.0/bin/
// 启动kafka
./kafka-server-start.sh -daemon ../config/server.properties &      //-daemon代表着以后台模式运行kafka集群，这样kafka的启动就不会影响我们继续在控制台输入命令。
//查看服务是否正常
jps

# 4、创建topic，测试
## 4.1创建topic
### 创建Topic
```
./kafka-topics.sh --create --zookeeper 10.0.0.60:2181 --replication-factor 2 --partitions 1 --topic shuaige
```
// 解释
```
--replication-factor 2   #复制两份
--partitions 1 #创建1个分区
--topic #主题为shuaige
```
'''在一台服务器上创建一个发布者'''

### 创建一个broker，发布者
```
./kafka-console-producer.sh --broker-list 10.0.0.60:9092 --topic shuaige
'''在一台服务器上创建一个订阅者'''
./kafka-console-consumer.sh --zookeeper localhost:2181 --topic shuaige --from-beginning
```

## 4.2 查看topic
```
./kafka-topics.sh --list --zookeeper localhost:2181
```

## 4.3 查看topic状态
```
/kafka-topics.sh --describe --zookeeper localhost:12181 --topic shuaige
```
下面是显示信息
Topic:ssports    PartitionCount:1    ReplicationFactor:2    Configs:
    Topic: shuaige    Partition: 0    Leader: 1    Replicas: 0,1    Isr: 1
//分区为为1  复制因子为2   他的  shuaige的分区为0
//Replicas: 0,1   复制的为0，1


## 4.4 创建kafka topic
```
bin/kafka-topics.sh --zookeeper node01:2181 --create --topic t_cdr --partitions 30 --replication-factor 2
```
注： partitions指定topic分区数，replication-factor指定topic每个分区的副本数
* partitions分区数:
    * partitions ：分区数，控制topic将分片成多少个log。可以显示指定，如果不指定则会使用broker(server.properties)中的num.partitions配置的数量
    * 虽然增加分区数可以提供kafka集群的吞吐量、但是过多的分区数或者或是单台服务器上的分区数过多，会增加不可用及延迟的风险。因为多的分区数，意味着需要打开更多的文件句柄、增加点到点的延时、增加客户端的内存消耗。
    * 分区数也限制了consumer的并行度，即限制了并行consumer消息的线程数不能大于分区数
    * 分区数也限制了producer发送消息是指定的分区。如创建topic时分区设置为1，producer发送消息时通过自定义的分区方法指定分区为2或以上的数都会出错的；这种情况可以通过alter –partitions 来增加分区数。
* replication-factor副本
    * replication factor 控制消息保存在几个broker(服务器)上，一般情况下等于broker的个数。
    * 如果没有在创建时显示指定或通过API向一个不存在的topic生产消息时会使用broker(server.properties)中的default.replication.factor配置的数量
查看所有topic列表
```
bin/kafka-topics.sh --zookeeper node01:2181 --list
```
查看指定topic信息
```
bin/kafka-topics.sh --zookeeper node01:2181 --describe --topic t_cdr
```
控制台向topic生产数据
```
bin/kafka-console-producer.sh --broker-list node86:9092 --topic t_cdr
```
控制台消费topic的数据
```
bin/kafka-console-consumer.sh --zookeeper node01:2181 --topic t_cdr --from-beginning
```
查看topic某分区偏移量最大（小）值
```
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --topic hive-mdatabase-hostsltable --time -1 --broker-list node86:9092 --partitions 0
```
注： time为-1时表示最大值，time为-2时表示最小值
增加topic分区数, 为topic t_cdr 增加10个分区
```
bin/kafka-topics.sh --zookeeper node01:2181 --alter --topic t_cdr --partitions 10
```
删除topic，慎用，只会删除zookeeper中的元数据，消息文件须手动删除
```
bin/kafka-run-class.sh kafka.admin.DeleteTopicCommand --zookeeper node01:2181 --topic t_cdr
```
查看topic消费进度
这个会显示出consumer group的offset情况， 必须参数为--group， 不指定--topic，默认为所有topic
```
Displays the: Consumer Group, Topic, Partitions, Offset, logSize, Lag, Owner for the specified set of Topics and Consumer Group
bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker
required argument: [group]
Option Description
------ -----------
--broker-info Print broker info
--group Consumer group.
--help Print this message.
--topic Comma-separated list of consumer
topics (all topics if absent).
--zkconnect ZooKeeper connect string. (default: localhost:2181)
Example,
bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --group pv
Group Topic Pid Offset logSize Lag Owner
pv page_visits 0 21 21 0 none
pv page_visits 1 19 19 0 none
pv page_visits 2 20 20 0 none
```
以上图中参数含义解释如下：
```
topic：创建时topic名称
pid：分区编号
offset：表示该parition已经消费了多少条message
logSize：表示该partition已经写了多少条message
Lag：表示有多少条message没有被消费。
Owner：表示消费者
```
细看kafka-run-class.sh脚本，它是调用 了ConsumerOffsetChecker的main方法，所以，我们也可以通过java代码来访问scala的ConsumerOffsetChecker类，代码如下：
```
import kafka.tools.ConsumerOffsetChecker;
/**
  * kafka自带很多工具类，其中ConsumerOffsetChecker能查看到消费者消费的情况,
  * ConsumerOffsetChecker只是将信息打印到标准的输出流中
  *
  */
public class RunClass {
    public static void main(String[] args) {
    //group-1是消费者的group名称,可以在zk中
    String[] arr = new String[]{"--zookeeper=192.168.199.129:2181,192.168.199.130:2181,192.168.199.131:2181/kafka","--group=group-1"};
    ConsumerOffsetChecker.main(arr);
    }
}
```


# 5、日志说明
默认kafka的日志是保存在/opt/kafka/kafka_2.10-0.9.0.0/logs目录下的，这里说几个需要注意的日志
server.log #kafka的运行日志
state-change.log #kafka他是用zookeeper来保存状态，所以他可能会进行切换，切换的日志就保存在这里
controller.log #kafka选择一个节点作为“controller”,当发现有节点down掉的时候它负责在游泳分区的所有节点中选择新的leader,这使得Kafka可以批量的高效的管理所有分区节点的主从关系。如果controller down掉了，活着的节点中的一个会备切换为新的controller.

# 6、 Kafka-manager（v2.0.0.2）（Kafka集群可视化管理）
## 6.1 sbt编译
```
curl https://bintray.com/sbt/rpm/rpm > bintray-sbt-rpm.repo
mv bintray-sbt-rpm.repo /etc/yum.repos.d/
yum install sbt
```
检查sbt是否安装成功
```
sbt version
```
## 6.2 安装部署kafka-manager
```
wget https://github.com/yahoo/kafka-manager/releases/kafka-manager-2.0.0.2.tar.gz
tar zxvf kafka-manager-2.0.0.2.tar.gz -C /usr/local
```
编译kafka-manager
```
cd /usr/local/kafka-manager-2.0.0.2
./sbt clean dist     //需要一段时间
```
编译结果查看
```
ls /usr/local/kafka-manager-2.0.0.2/target/universal/   //存在kafka-manager-2.0.0.2.zip
```
# 创建目录kafka-manager
```
mkdir -p /usr/local/kafka-manager
cp /usr/local/kafka-manager-2.0.0.2/target/universal/kafka-manager-2.0.0.2.zip /usr/local/kafka-manager
```
解压文件
```
unzip kafka-manager-2.0.0.2.zip
```
修改配置文件
```
vim /usr/local/kafka-manager/kafka-manager-2.0.0.2/conf/application.conf
```
修改信息
```
单机：
kafka-manager.zkhosts="localhost:2181"
集群：
kafka-manager.zkhosts="node3.cn:2181,node4.cn:2181,node5.cn:2181"
```
## 6.3启动kafka-manager
控制台启动
```
bin/kafka-manager
```
后台守护启动
```
nohup bin/kafka-manager &
```
后台启动通过 -Dhttp.port，指定端口; -Dconfig.file=conf/application.conf指定配置文件
```
nohup bin/kafka-manager -Dconfig.file=conf/application.conf -Dhttp.port=8080 &
```



