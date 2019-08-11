---
title: Linux命令相关
date: 2019-07-28 00:13:03
tags: linux
categories: 操作系统
---

关于linux操作系统的一些使用命令，看下面。

<!-- more -->
![linux](Linux命令相关/linux.jpeg)

# 1. linux下查看某个文件或文件夹占用的磁盘空间大小
```
du -ah --max-depth=1
```

# 2.sed修改文件
在每行行首或者行尾添加相同的字符串
```
sed 's/^/HEAD&/g' text.file    每行行首添加HEAD
sed 's/$/&TAIL/g' text.file    每行行尾添加TAIL
```
如果要修改原文件，则添加 -i参数
```
sed -i 's/^/HEAD&/g' text.file
sed -i 's/$/&TAIL/g' text.file
```
递归替换
```
find . -type f -print0 | xargs -0 sed -i 's/10.1.0.33,10.1.0.44,10.1.0.48/${es_nodes}/g'
```
文件第一行添加字符串
```
sed -i "1i\添加内容" filename
```

# 3.查看centos版本
```
cat /etc/redhat-release
```

# 4.查看cpu
```
cat /proc/cpuinfo |grep "physical id"|sort|uniq|wc -l       查看cpu核数
cat /proc/cpuinfo | grep "cpu cores" | uniq             	物理cpu个数
```

# 5.查看内存
```
free -h
```

# 6.查看磁盘容量
```
df -h
```

# 7.查看端口号对应进程号
```
netstat -tunlp|grep 端口号
```

# 8.查看未释放空间的进程
```
lsof | grep deleted     
```

# 9.杀死未释放空间的进程
```
lsof | grep deleted | awk '{print $2}' | sort | uniq | xargs kill -9
```

# 10.grep
```
grep -o "ods\.[a-z|A-Z|_]*" ods2report.py | grep "_" | sort | uniq -c
grep -o只显示匹配内容
uniq -c计算重复行数量
grep "\"db\"" *.py | awk -F ':' '{print $1}' | sort | uniq
grep -o "ods\.[a-z|A-Z|_|0-9]*" *.py | awk -F ':' '{print $2}' |grep "_" | sort | uniq -c
```

# 11.查看详细进程信息
```
top -c
```

