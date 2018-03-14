---
title: Windows远程连接linux图形界面配置详解
date: 2017-12-28 17:30:16
tags: [windows,linux,vnc]
categories: 操作系统
---
Windows远程连接linux图形界面，利用VNC服务实现windows远程连接linux图形化界面，linux作为VNC Server，windows作为VNC Viewer。原理很简单，在vnc server端生成一个桌面号，在vnc client端去连接该桌面号即可。其中很神奇的地方在于，如果两个人同时连接上一个桌面号的话，一个人可以看到另一个人的操作。
<!-- more -->
安装步骤
# mini版centos安装图形化界面
如果已经安装了图形化界面，则此步骤可以省略。
## 安装X window
```
yum groupinstall "X Window System" 
```
## 安装GNOME Desktop
```
yum groupinstall "GNOME Desktop"
```
如果是centos7以前的版本，则安装命令为
```
yum groupinstall "Desktop"
```
如果找不到Desktop，那么试试：
```
yum grouplist
```
查看可以安装的group，可能不同的版本group组的名字不同。
## 启动gnome
```
startx
```
切换到图形化界面
# linux安装VNC Server
## 安装
```
yum install vnc-server –y
```
## 配置
```
cp /lib/systemd/system/vncserver@.service /etc/systemd/system/
cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
```
编辑vim /etc/systemd/system/vncserver@:1.service将<USER>更换为root
## 设置开机自启动
```
systemctl enable vncserver@:1.service
```
## 添加防火墙信任规则
```
firewall-cmd --permanent --add-service vnc-server
firewall-cmd –reload
```
## 重启服务器reboot
## 启动vnc服务
启动方式：
```
vncserver :桌面号
```
注意：中间需留有空格，桌面号用数字表示，表示每个用户占用一个桌面连接。
以上命令执行的过程中，因为是第一次执行，需要输入密码，密码被加密/root/.vnc/passwd中；同时在用户主目录下的.vnc子目录中为用户自动建立xstartup配置文件（/root/.vnc/xstartup），在每次启动VND服务时，都会读取该文件中的配置信息。 
## VNC服务使用的端口号与桌面号的关系
![桌面号与端口号的关系](Windows远程连接linux图形界面配置详解\deskNumMappingPort.png)
# windows安装VNC Viewer
## 安装
## 测试
![连接测试](Windows远程连接linux图形界面配置详解\vnc_viewer_connect_test.png)
输入ip:桌面号连接
![连接成功界面](Windows远程连接linux图形界面配置详解\vnc_viewer_connect_appearance.png)
# 其他
## 修改vnc密码
```
vncpasswd
```
## 关闭vnc服务
```
vncserver -kill :1
```
## 防火墙添加信任
```
firewall-cmd --permanent --add-service vnc-server
firewall-cmd --reload
```
