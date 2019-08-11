---
title: windows下gvim打开文件中文乱码
date: 2018-02-08 14:36:41
tags: [windows,gvim]
categories: 操作系统
---
习惯了linux操作命令，突然有需要使用到windows cmd命令，例如安装某些软件时，需要用命令方式去安装：
```
npm install hexo-cli
```
安装完成后，需要编辑一些配置文件，这个时候，去'计算机'中重新定位到该配置文件的位置时，是很不方便的，如果有个类似vim的工具多好，windows自带的文本编辑工具notepad打开后还不能像vim一样操作，很是不适，不过总有神一般的人物开发出好用的工具。
windows下有类似linux下的vim工具gvim，但是gvim打开某些文件时，中文乱码，很是让人烦恼，下面就来介绍如何解决乱码的问题。
<!-- more -->
windows下默认vim打开是gbk格式的，所以中文乱码，需要进行设置vim打开时加载文件时的编码，参照如下设置：
# 打开gvim客户端
![gvim客户端](windows下gvim打开文件中文乱码\gvim客户端.png)
# 编辑_vimrc配置文件
## 方式一：
![gvim查找](windows下gvim打开文件中文乱码\gvim查找.png)
![编辑_vimrc文件](windows下gvim打开文件中文乱码\_vimrc.png)
## 方式二：直接编辑文件%VIM_HOME%\_vimrc
# 添加如下配置：
```
set enc=utf8 设置打开文件缓冲区编码
set fencs=utf8,gbk,gb2312,gb18030,cp936	设置文件编码
```
设置后，再次打开ok。
如果gvim菜单栏中文乱码
编辑配置文件_vimrc，添加如下配置：
```
source $VIMRUNTIME/delmenu.vim	设置gvim菜单文件编码
source $VIMRUNTIME/menu.vim	设置gvim菜单文件编码
```