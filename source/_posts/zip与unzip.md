---
title: zip与unzip
date: 2019-12-06 00:10:49
tags: [Linux, zip, unzip]
categories: Linux
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux或者mac命令zip和unzip使用频率还是蛮高的，下面介绍下如何打zip包与解压zip包。

<!-- more -->

# 1.打包
```
zip -q -r -m -o a.zip a/
    -q   不显示压缩进度
    -r   子目录子文件也包含到zip中（很重要）
    -e   加密
    -m   压缩完，删除源文件夹
```

# 2.解压
```
unzip a.zip 即可解压
unzip "*.zip”：解压目录下的多个zip文件
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果解压目录下所有的zip包，使用下面命令：
```
unzip *.zip
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;会报错，因为它会去第一个解压后的目录中查找第二个zip，第二个解压的zip目录中查找第三个解压的zip，所以会出现filename not matched问题。
```
caution: filename not matched: a.zip
```

- - -
<b>Where there is an enemy, there is a friend.</b>