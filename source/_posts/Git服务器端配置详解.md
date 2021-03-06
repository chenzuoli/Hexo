---
title: Git服务器端配置详解
date: 2018-03-10 13:01:09
tags: [git]
categories: git
---

我们在公司中，一个项目在开发过程中必定要涉及到同事之间的协同作战，此时代码管理就必不可少了，程序员用的最多的就是git了吧，但是公司的代码是禁止上传到github上的，所以需要自己搭建一个内部的git server服务器供公司内部使用，下面来具体就介绍。
<!-- more -->
本安装教程适用于centos服务器，其他版本linux服务器步骤一样，运行命令会不相同，自己转换即可。
配置git server，原理就是将客户端的公钥id_rsa.pub添加到服务端的密钥文件authorized_keys中，多个用户另起一行拼接到该文件中即可。下面介绍安装的两种方法：
# 方法一：yum安装
## 安装git软件
```
yum install git -y
git --version
```
git --version可以查看安装的git版本
```
[git@hadoop3 gitrepo]$ git --version
git version 1.8.3.1
```
系统是centos7，自带git版本太低，但可以使用。
## 添加git用户组和用户
```
groupadd git
useradd git -g git
```
## 创建登录证书
```
su git
cd
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```
这里注意，一定要设置authorized_keys文件的权限为600，权限太大或太小都会造成cannot access的问题，我就遇到过这个坑。
## 免秘钥
复制客户端的公钥到服务端的authorized_keys文件中
## 初始化git仓库
创建一个git仓库文件夹，专门存放项目的地方，并创建一个项目，初始化：
```
mkdir /srv/git -p
cd /srv/git
mkdir project.git
cd project.git
git init --bare
```
bare的意思就是初始化一个裸仓库，并不存储用户push的数据，只存储元数据。
## 提交项目到git server
下面的命令是在客户端（另一台机器，可以是windows，也可以是linux）上执行的：
```
cd myproject
git init
git config --global user.email “chenzuoli709@163.com”
git config --global user.name “chenzuoli”
git add .
git commit -m “initial commit”
git remote add origin git@gitserver:/srv/git/project.git
git push origin master
```
注意：提交的时候需要将gitserver更换成自己的git服务器的ip或者映射域名。
## 克隆项目
```
git clone git@gitserver:/srv/git/project.git
```
也是要替换gitserver的。
# 方法二：自定义安装
## 下载最新稳定版git
<a href="https://www.kernel.org/pub/software/scm/git/git-2.9.5.tar.xz">git最新版下载</a>
## 添加git用户组和用户
```
groupadd git
useradd git -g git
```
## 上传解压
```
su git
cd $GIT_HOME
xz -d git-2.9.5.tar.xz
tar xvf git-2.9.5.tar
```
## 环境准备
```
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel gcc -y
```
## 预编译、编译、安装
```
cd git-2.9.5
./configure -prefix=$GIT_HOME
make && make install
```
其中GIT_HOME是自己指定的git安装目录。
## 配置
配置跟方法一一样
## 添加链接
上述步骤配置完成后，我们在push clone时会出现如下问题：
```
bash: git-receive-pack: command not found
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.
bash: git-upload-pack: command not found
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.
```
原因是自定义安装时git的执行文件不在/usr/bin下，添加链接即可：
```
ln -s /usr/local/gitserver/install/bin/git /usr/bin/git
ln -s /usr/local/gitserver/install/bin/git-shell /usr/bin/git-shell
ln -s /usr/local/gitserver/install/bin/git-upload-archive /usr/bin/git-upload-archive
ln -s /usr/local/gitserver/install/bin/git-upload-pack /usr/bin/git-upload-pack
ln -s /usr/local/gitserver/install/bin/git-receive-pack /usr/bin/git-receive-pack
```
到这里配置就基本完成了，谢谢大家，如果有什么问题，请留言。