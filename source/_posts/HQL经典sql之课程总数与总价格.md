---
title: HQL经典sql之课程总数与总价格
date: 2019-10-26 11:06:40
tags: [Hive,SQL,数据分析]
categories: 数据仓库
notebook: 数据仓库
---

有这样一个问题，关于培训班学生的统计分析数据，一起来看看吧。

<!-- more -->

有三张表：
- t1为学生课程表：sid（学生ID）、courseid（课程ID）、date（报课时间）
- t2为学生基本信息表：sid、name、age......
- t3为课程基本信息表：courseid、price、date
现在想要查看一个培训学校目前每个学生的所报的所有课程数和花费的总金额，如何求呢？
```
select
    sid,
    count(courseid) over(partition by sid) as count,
    sum(price) over(partition by sid) as sumprice
from (
    select
        sid, t1.courseid as courseid, price
    from t1, t3
    where t1.courseid = t3.courseid
) a;
```
分析：
1. 首先关联出学生所报课程的价格；
```
select
    sid, t1.courseid as courseid, price
from t1, t3
where t1.courseid = t3.courseid
```
2. 使用开窗函数对sid分组得到学生报的每门课程和没门课程对应的价格，然后使用count和sum求得想要的指标；


- - -
<b>I wish i had two dogs, one would be given to my parents, one would be with me.</b>