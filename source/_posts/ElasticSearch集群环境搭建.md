---
title: ElasticSearch集群环境搭建
date: 2019-09-06 20:23:45
tags: ElasticSearch
categories: 环境搭建
---

ElasticSearch作为一个基于搜索引擎lucene的文档数据库，搜索速度在目前的大数据存储系统中，算是佼佼者了。
又有许多人把它当数据库使用，索引做库，类型做表，也是不错的选择。
在数据仓库中，也可以做report层的存储，对接数据可视化工具，提供接口查询业务数据、结果数据。
下面来看看集群环境怎么搭建。
![es.logo](ElasticSearch集群环境搭建/es.jpeg)
<!-- more -->

# 1、软件需求
jdk8
elasticsearch包
# 2、es安装、配置
```
tar -zxvf elasticsearch-*.tar.gz -C /usr/local
cd /usr/local/elasticsearch
vim /config/elasticsearch.yml
cluster.name: es-cluster-1             配置集群名称 三台服务器保持一致
node.name: node-1                      配置单一节点名称，每个节点唯一标识
network.host: 0.0.0.0                  设置绑定的ip地址
http.port: 9200                        端口
discovery.zen.ping.unicast.hosts: ["192.168.21.12", "192.168.21.13","192.168.21.14"]   集群节点ip或者主机
discovery.zen.minimum_master_nodes: 3    设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）
```
## 2.1新建用户（三台都需要）
在 Linux 环境中，elasticsearch 不允许以 root 权限来运行！所以需要创建一个非root用户，以非root用户来起es
elsearch                                             新增elsearch用户组
```
useradd elsearch -g elsearch -p elasticsearch                 创建elsearch用户
chown -R elsearch:elsearch ./elasticsearch                    用户目录权限
```
## 2.2运行操作三台服务
```
su elsearch
./bin/elasticsearch -d           //-d表示后台启动
```
# 3、问题汇总解决方案
## 3.1问题一
```
ERROR: bootstrap checks failed
max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536]
```
原因：无法创建本地文件问题,用户最大可创建文件数太小
解决方案：
切换到root用户，编辑limits.conf配置文件， 添加类似如下内容：
```
vi /etc/security/limits.conf
```
添加如下内容:
```
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```
备注：* 代表Linux所有用户名称（比如 hadoop）
保存、退出、重新登录才可生效
## 3.2问题二
```
max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
```
原因：最大虚拟内存太小
解决方案：切换到root用户下，修改配置文件sysctl.conf
```
vi /etc/sysctl.conf
```
添加下面配置：
```
vm.max_map_count=655360
```
并执行命令：
sysctl -p
然后重新启动elasticsearch，即可启动成功。
## 3.3问题三：
```
max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
```
原因：最大虚拟内存太小
解决方案：切换到root用户下，修改配置文件sysctl.conf
vi /etc/sysctl.conf
添加下面配置：
```
vm.max_map_count=655360
```
并执行命令：
sysctl -p
然后重新启动elasticsearch，即可启动成功。
## 3.4问题四：
ElasticSearch启动找不到主机或路由
原因：ElasticSearch 单播配置有问题
解决方案：
检查ElasticSearch中的配置文件
vi  config/elasticsearch.yml
找到如下配置：
discovery.zen.ping.unicast.hosts: ["192.168.21.12", "192.168.21.13","192.168.21.14"]  
一般情况下，是这里配置有问题，注意书写格式
## 3.5问题五：
org.elasticsearch.transport.RemoteTransportException: Failed to deserialize exception response from stream
原因:ElasticSearch节点之间的jdk版本不一致
解决方案：ElasticSearch集群统一jdk环境
## 3.6问题六：
Unsupported major.minor version 52.0
原因：jdk版本问题太低
解决方案：更换jdk版本，ElasticSearch5.0.0支持jdk1.8.0


