---
title: OLAP与OLTP的区别
date: 2019-12-23 23:38:43
tags: [OLAP,OLTP,数据仓库]
categories: 数据仓库
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OLAP: 联机分析处理，作为一种多维查询和分析的工具，是数据仓库功能的自然扩展；
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OLTP: 联机事务处理

<img src="OLAP与OLTP的区别/difference.jpeg" width="500" height="300"/>

<!-- more -->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OLAP与OLTP的区别：

比较项 | OLAP | OLTP
:-: | :-: | :-:
特性 | 信息处理 | 操作处理
用户 | 面向决策人员 | 面向操作人员
功能 | 支持管理需要 | 支持日常操作
面向 | 面向数据分析 | 面向应用
驱动 | 分析驱动 | 事务驱动
数据量 | 一次处理的数据量大 | 一次处理的数据量小
访问 | 不可更新，但周期性刷新 | 可更新
数据 | 历史数据 | 当前值数据
汇总 | 综合性和提炼性数据 | 细节性数据
视图 | 导出数据 | 原始数据

- - -
<b>不要试图去改变别人，做好自己应该做的事情，多思考为什么，大胆询问，大胆尝试。</b>
