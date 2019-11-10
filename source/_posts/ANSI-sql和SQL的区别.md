---
title: ANSI-sql和SQL的区别
date: 2019-11-10 09:54:15
tags: [SQL,ANSI-SQL]
categories: 数据仓库
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;相信大家都使用过SQL SERVER。今天给大家简单介绍一下Oracle SQL与ANSI SQL区别。其实，SQL SERVER与与ANSI SQL也有区别。

<img src="ANSI-sql和SQL的区别/structure_query_language.jpeg" width="500" height="300"/>

<!-- more -->

1. <b>什么是ANSI</b>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ANSI：美国国家标准学会（American National Standards Institute）。当时，美国的许多企业和专业技术团体，已开始了标准化工作，但因彼此间没有协调，存在不少矛盾和问题。为了进一步提高效率，数百个科技学会、协会组织和团体，均认为有必要成立一个专门的标准化机构，并制订统一的通用标准。

2. <b>ANSI SQL到底是什么</b>
  （1）作为程序员开发者们应该知道，在使用那些非标准的SQL命令（比如Oracle、微软和MySQL等数据库系统）从跨平台和遵守标准的角度出发，你应该尽量采用ANSI SQL，它是一种和平台无关的数据库语言。其实为什么这么说了，很简单就是可能在Oracle能够运行的SQL语句不一定在SQL SERVER当中能够运行，那么在跨平台当中数据操作就会带来困难。
 （2）程序在开发的时候，如果使用SQL语句对数据进行操作。一般的建议不管你在使用哪种数据库系统，如果该数据库系统中的SQL完全支持ANSI SQL标准，那么请你尽量使用ANSI SQL。

3. <b>一些写法区别</b>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;既然标准不一样，那么写法肯定有一些区别，具体可以参考这2篇文章(Presto查询系统使用的是ANSI-SQL语法)：
> <a>[<font color=#0099ff>Presto-SQL与Hive-SQL的区别与联系</font>](http://wetech.top/2019/11/09/Presto-SQL%E4%B8%8EHive-SQL%E7%9A%84%E5%8C%BA%E5%88%AB%E4%B8%8E%E8%81%94%E7%B3%BB/)</a>
<a>[<font color=#0099ff>Presto基础用法介绍</font>](https://note.youdao.com/ynoteshare1/index.html?id=1115e26a8ed49d1c34a7a37c3013a200&type=note)</a>


- - -
<b>No one can take away who you are.</b>