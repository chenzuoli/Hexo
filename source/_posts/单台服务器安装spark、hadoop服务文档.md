---
title: 单台服务器安装spark、hadoop服务文档
tags: [spark,hadoop]
categories: 大数据
date: 2017-12-26 18:16:51
---
spark作为分布式计算引擎，如果内存足够，是需要很少的磁盘空间的，在shuffle可能用到，在reduce阶段一定会用到，它是基于hdfs作为存储介质的，所以在使用spark时，应该搭建一个hdfs。
<!-- more -->
# 安装JDK1.8
安装并配置环境变量，步骤略。
# 安装scala2.11.8
安装并配置环境变量，步骤略。
# hadoop伪分布式搭建
## 关闭防火墙
## 配置本机对本机免秘钥登录
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
ssh-copy-id ip
其中ip为本机ip
Ssh ip
首次本机ssh本机需要输入密码或者yes，输入即可，第二次或者以后就不需要输入参数了。
## 下载hadoop-2.7.4.tar.gz包
## 解压
## 修改配置文件HADOOP_HOME/etc/hadoop下
### Hadoop.env.sh
修改JAVA_HOME为jdk路径；

### Core-site.xml
Fs.defaltFS属性修改为namenode的ip
Hadoop.tmp.dir修改为自定义目录，并创建好该目录
```
<property>
  	<name>fs.defaultFS</name>
  	<value>hdfs://192.168.109.235:9000</value>
</property>
<property>
  	<name>hadoop.tmp.dir</name>
  	<value>/root/chen/hadoop/data/temp</value>
</property>
<property>
  	<name>fs.trash.interval</name>
  	<value>1440</value>
</property>
```
### Hdfs-site.xml
使用默认值即可
```
<property>
    <name>dfs.replication</name>
    <value>2</value>
</property>
<property>
    <name>dfs.permissions</name>
    <value>false</value>
</property>
```
### Mapred-env.sh
修改JAVA_HOME为jdk路径，其他默认。

### Mapred-site.xml
```
<property>
   	<name>mapreduce.framework.name</name>
   	<value>yarn</value>
</property>
```
### yarn-env.sh
修改JAVA_HOME为java安装路径

### yarn-site.xml
```
yarn.resourcemanager.hostname属性指定为namenode的ip地址。
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>
<property>
   	<description>The hostname of the RM.</description>
   	<name>yarn.resourcemanager.hostname</name>
   	<value>192.168.109.235</value>
</property>
<property>
	<name>yarn.nodemanager.resource.memory-mb</name>
	<value>2048</value>
</property>
```
### 添加slaves文件
在HADOOP_HOME/etc/hadoop文件夹下添加slaves文件，指定datanode节点
添加localhost即可。
	
## 格式化namenode
./bin/hdfs namenode –format

## 启动hdfs
./sbin/start-all.sh

## jps查看节点服务的启动情况
如果启动正常，那么应该有
Namenode
SecondaryNamenode
Resourcemanager
Nodemanager
DataNode
这5个角色
Web Ui访问：http://ip:50070
---
# Spark搭建
## 下载并解压spark-2.1.0-bin-hadoop2.7.tgz
## 修改配置文件
### cp slaves.template slaves
### cp spark-env.sh.template spark-env.sh
### cp spark-defaults.conf.template spark-defaults.conf
### vi spark-env.sh
增加参数
```
SPARK_MASTER_HOST=修改为ip
SPARK_MASTER_PORT=7077
SPARK_WORKER_CORES=2
SPARK_WORKER_MEMORY=4g
SPARK_WORKER_INSTANCES=3
HADOOP_CONF_DIR=/chen/hadoop2.7/hadoop-2.7.4/etc/hadoop修改为hadoop配置文件的位置
SPARK_DRIVER_MEMORY=1024M
JAVA_HOME=/chen/jdk8/jdk1.8.0_144修改为jdk的路径
MAVEN_OPTS="-Xms1024m -Xmx4096m -XX:PermSize=1024m"
```
### vi spark-deafults.conf
![spark-default.conf](单台服务器安装spark、hadoop服务文档\spark-default.conf.png)
其中需要修改hdfs的ip地址，并创建路径/user/spark/logs
## 启动spark
./sbin/start-all.sh
正常启动的话应该有：
1个Master
3个Worker
两个角色
Web Ui访问http://ip:8080

