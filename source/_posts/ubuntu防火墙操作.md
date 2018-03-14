---
title: ubuntu防火墙操作
date: 2018-02-07 19:03:40
tags: [ubuntu,防火墙]
categories: ubuntu
---
使用过了centos的同胞们，听说ubuntu的交互性很不错，可视化界面也很炫酷，果断更换ubuntu系统，但是安装完成之后，感觉都不会使用linux系统了，于是各种google查询操作方法，下面来简单介绍ubuntu防火墙的操作。
<!-- more -->
使用ubuntu系统，配置防火墙稍微跟centos不太一样，有一样工具，叫做ufw，即uncomplicated firewall简单防火墙，刚开始用的时候不太习惯，记住这两个单词就行。
ubuntu系统自带就有这个工具，可能版本的原因，你的ubuntu可能没有，不用担心，没有先来安装。
# 安装ufw工具
```
sudo apt install ufw -y
```
如果报错找不到包：
```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package ufw
```
更新一下依赖库就行：
```
sudo apt-get update
```
# 然后继续安装ufw，安装完成后，我们来启动它
```
sudo ufw enable
```
# 此时防火墙就开启了，默认可以访问部分端口，不如22、443，想关闭所有外部ip对本机的端口访问的话，执行命令：
```
sudo ufw default deny
```
# 查看防火墙状态
```
sudo ufw status
```
# 启用或者禁用端口、服务
## 允许外部访问端口
```
sudo ufw allow 22
sudo ufw allow sshd
```
## 禁止外部访问端口
```
sudo ufw delete allow 80
sudo ufw delete allow apache2
```
## 允许某个ip访问本机所有端口
```
sudo ufw allow from 192.168.1.1
```
OK，希望对大家有帮助，我们一起进步，有问题欢迎在下方留言，或者给我发邮件，邮件地址：chenzuoli709@gmail.com。