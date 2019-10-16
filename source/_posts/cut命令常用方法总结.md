---
title: cut命令常用方法总结
date: 2019-10-16 16:26:39
tags: [cut, linux]
categories: Linux
notebook: 笔记
---

Linux：cut命令详解，cut能干什么呢？
1. 文件内容查看；
2. 显示行中的指定部分，删除文件中指定字段；
3. 显示文件的内容，类似于下的type命令；

<img src="cut命令常用方法总结/cut.jpeg" width="500" height="150"/>

<!-- more -->

# 1.说明
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该命令有两项功能，其一是用来显示文件的内容，它依次读取由参数file所指明的文件，将它们的内容输出到标准输出上；其二是连接两个或多个文件，如cut fl f2 > f3将把文件fl和几的内容合并起来，然后通过输出重定向符“>”的作用，将它们放入文件f3中。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当文件较大时，文本在屏幕上迅速闪过（滚屏），用户往往看不清所显示的内容。因此，一般用more等命令分屏显示。为了控制滚屏，可以按Ctrl+S键，停止滚屏；按Ctrl+Q键可以恢复滚屏。按Ctrl+C（中断键可以终止该命令的执行，并且返回Shell提示符状态。

# 2.语法
cut(选项)(参数)
## 2.1 选项
```
-b：仅显示行中指定直接范围的内容；
-c：仅显示行中指定范围的字符；
-d：指定字段的分隔符，默认的字段分隔符为“TAB”；
-f：显示指定字段的内容；
-n：与“-b”选项连用，不分割多字节字符；
--complement：补足被选择的字节、字符或字段；
--out-delimiter=<字段分隔符>：指定输出内容是的字段分割符；
--help：显示指令的帮助信息；
--version：显示指令的版本信息；
```

## 2.2 参数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;文件：指定要进行内容过滤的文件。

# 3.实例
## 3.1.例如有一个学生报表信息，包含No、Name、Mark、Percent：
```
cat test.txt

No Name Mark Percent
01 tom 69 91
02 jack 71 87
03 alex 68 98
```

使用 -f 选项提取指定字段：
```
cut -f 1 test.txt
```
结果：
```
No
01
02
03
```

```
cut -f2,3 test.txt
```
结果：
```
Name Mark
tom 69
jack 71
alex 68
```

--complement 选项提取指定字段之外的列（打印除了第二列之外的列）：
```
cut -f2 --complement test.txt
```
结果：
```
No Mark Percent
01 69 91
02 71 87
03 68 98
```

## 3.2.使用 -d 选项指定字段分隔符：
```
cat test2.txt

No;Name;Mark;Percent
01;tom;69;91
02;jack;71;87
03;alex;68;98
```
```
cut -f2 -d";" test2.txt
```
结果：
```
Name
tom
jack
alex
```

## 3.3.指定字段的字符或者字节范围
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cut命令可以将一串字符作为列来显示，字符字段的记法：
```
N-：从第N个字节、字符、字段到结尾；
N-M：从第N个字节、字符、字段到第M个（包括M在内）字节、字符、字段；
-M：从第1个字节、字符、字段到第M个（包括M在内）字节、字符、字段；
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面是记法，结合下面选项将摸个范围的字节、字符指定为字段：
```
-b 表示字节；
-c 表示字符；
-f 表示定义字段；
```

# 4.例：
```
cat test.txt

abcdefghijklmnopqrstuvwxyz
abcdefghijklmnopqrstuvwxyz
abcdefghijklmnopqrstuvwxyz
abcdefghijklmnopqrstuvwxyz
abcdefghijklmnopqrstuvwxyz
```

打印第1个到第3个字符：
```
cut -c1-3 test.txt
```
结果：
```
abc
abc
abc
abc
abc
```

打印前2个字符：
```
cut -c-2 test.txt
```
结果：
```
ab
ab
ab
ab
ab
```

打印从第5个字符开始到结尾：
```
cut -c5- test.txt
```
结果：
```
efghijklmnopqrstuvwxyz
efghijklmnopqrstuvwxyz
efghijklmnopqrstuvwxyz
efghijklmnopqrstuvwxyz
efghijklmnopqrstuvwxyz
```

- - -
You need to know what you need.