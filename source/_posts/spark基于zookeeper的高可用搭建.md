---
title: spark基于zookeeper的高可用搭建
tags: [spark,zookeeper]
date: 2017-12-26 21:26:15
categories: 大数据
---
spark提供服务时，master的角色非常的重要，它负责任务分发、任务调度，可谓任重道远啊，所以我们要对master做高可用，基于zookeeper的高可用，可以自动实现master挂掉后备用的master启动，堆外提供服务。
<!-- more -->
# 节点分布：
zookeeper: node1 node2 node3
spark: node1 node2 node3 node4
# 编辑SPARK_HOME/conf/spark-env.sh
注释掉HADOOP_CONF_DIR，添加SPARK_DAEMON_JAVA_OPTS，其他配置不变。
```
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=node1:2181,node2:2181,node3:2181 -Dspark.deploy.zookeeper.dir=/spark20170302"
```
# 同步该配置文件spark-env.sh到其他节点
scp spark-env.sh root@node2:$SPARK_HOME/conf
scp spark-env.sh root@node3:$SPARK_HOME/conf
scp spark-env.sh root@node4:$SPARK_HOME/conf
# 在node2节点上编辑spark-env.sh，将SPARK_MASTER_HOST修改为node2
```
SPARK_MASTER_HOST=node2
```
# 启动spark服务
在node1节点上启动spark集群
```
$ ./sbin/start-all.sh
```
# 启动另一个master
在node2节点上只启动master
```
$ ./sbin/start-master.sh
```
# 访问webUI查看启动情况
如果配置正确，启动正常，那么master会有两个（node1， node2），一个为ACTIVE状态，一个为STANDBY状态。

