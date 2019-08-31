---
title: 搭建zookeeper集群
date: 2019-08-31 10:46:45
tags: zookeeper
categories: 环境搭建
---

zookeeper作为一个hadoop生态组件的连接器，在节点服务之间的通信及元数据管理上起着非常重要的作用，下面看看搭建步骤。
![zookeeper_logo](搭建zookeeper集群/zookeeper_small.gif)
<!-- more -->

# 1、软件环境
    Linux服务器。使用数量为一台，三台，五台，（2*n+1）。zookeeper集群的工作是超过半数才能对外提供服务，三台中超过两台超过半数，允许一台挂掉。最好不要使用偶数台。
例如：如果有4台，那么挂掉一台还剩下三台，如果再挂掉一台就不能行了，因为是要超过半数。
    Java jdk1.8
    zookeeper包
# 2、配置与安装zookeeper
## 配置文件zoo.cfg
```
tar -zxvf zookeeper-*.tar.gz  -C /usr/local
cd  /usr/local/zookeeper/conf
cp  zoo_sample.cfg zoo.cfg
vim zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper/zkdata
dataLogDir=/usr/local/zookeeper/zkdatalog
clientPort=2181
// 此处的IP就是你所操作的三台虚拟机的IP地址，每台虚拟机的zoo.cfg中都需要填入这三个地址。第一个端口是master和slave之间的通信端口，默认是2888，第二个端口是leader选举的端口，集群刚启动的时候选举或者leader挂掉之后进行新的选举的端口默认是3888
server.1=192.168.172.10:2888:3888
server.2=192.168.172.11:2888:3888
server.3=192.168.172.12:2888:3888
// server.1 这个1是服务器的标识也可以是其他的数字， 表示这个是第几号服务器，用来标识服务器，这个标识要写到快照目录下面myid文件里
创建myid文件。以现在所在的第一台虚拟机192.168.172.10为例，对应server.1，通过上边的配置信息可以查到。创建myid文件的目的是为了让zookeeper知道自己在哪台服务器上，例如现在所在的虚拟机是192.168.172.10，它对应的id是1，那么就在myid文件中写入1.
```

## 节点id配置
```
echo "1" > /usr/local/zookeeper/zkdata/myid
另外两台虚拟机上也需要创建myid文件并写入相应的id，id根据zoo.cfg文件中的IP地址查询。
echo "2" > /usr/local/zookeeper/zkdata/myid
echo "3" > /usr/local/zookeeper/zkdata/myid
```
# 3、启动zookeeper
```
cd /usr/local/zookeeper/bin
// 启动服务 (注意！三台虚拟机都要进行该操作)
./zkServer.sh start
// 检查服务器状态
./zkServer.sh status
```
// 显示如下
JMX enabled by default
Using config: /opt/zookeeper/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: follower #他是主节点leader还是从节点follower
# 4、补充说明
## 1. myid文件和server.myid
在快照目录下存放的标识本台服务器的文件，他是整个zk集群用来发现彼此的一个重要标识，myid必须与zoo.cfg配置中的 server.? 一致。
## 2. zoo.cfg配置文件
zoo.cfg文件是zookeeper配置文件，在conf目录里。
```
// tickTime：
这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
// initLimit：
这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 5个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 52000=10 秒
// syncLimit：
这个配置项标识 Leader 与Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是52000=10秒
// dataDir：
快照日志的存储路径
// dataLogDir：
事物日志的存储路径，如果不配置这个那么事物日志会默认存储到dataDir制定的目录，这样会严重影响zk的性能，当zk吞吐量较大的时候，产生的事物日志、快照日志太多
// clientPort：
这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。修改他的端口改大点
```