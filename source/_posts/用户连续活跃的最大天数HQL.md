---
title: 用户连续活跃的最大天数HQL
date: 2019-10-23 12:27:42
tags: [HQL]
categories: 数据仓库
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最近经常碰到这样的问题，每天每个城市播放最多的10首歌，某月每支股票连续下跌/上涨的最大天数，用户连续活跃的最大天数，初步看起来都和分析函数相关，考验逻辑思维和写复杂SQL的能力。

<img src="用户连续活跃的最大天数HQL/open.jpeg" width="500" height="150"/>

<!-- more -->

# 一、用户连续活跃的最大天数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以Oracle的分析函数语法说明，同理hive sql，首先模拟一些用户活跃的数据：
```
-- 建表语句
DROP TABLE sigin;
create table sigin(
    userid int, 
    sigindate varchar2(20) 
); 

-- 模拟数据插入
insert into sigin values(1,'2017-01-01');
insert into sigin values(1,'2017-01-02');
insert into sigin values(1,'2017-01-03');
insert into sigin values(1,'2017-01-04');
insert into sigin values(2,'2017-01-01');
insert into sigin values(2,'2017-01-02');
insert into sigin values(2,'2017-01-03');

insert into sigin values(1,'2017-01-10');
insert into sigin values(2,'2017-01-10');
insert into sigin values(1,'2017-01-11');
insert into sigin values(2,'2017-01-11');
insert into sigin values(1,'2017-01-12');
insert into sigin values(2,'2017-01-12');
```

大体思路如下：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先根据userid进行分组，将用户所有活跃的记录依次按照时间排序并标上序号。根据时间有序的特点，将所有时间减去它对应的序号，获取连续活跃时间唯一的时间点。如
```
2017-01-01 1
2017-01-02 2
2017-01-03 3
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;时间减去序号，得唯一时间2016-12-31。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据userid和这个连续活跃时间唯一的时间点进行分组，计算连续活跃天数。
```
-- 每个用户的几段连续活跃的天数
select 
    userid,
    to_date(sigindate,'yyyy-mm-dd')-sigin_rank as date_rank,
    count(1) as sigincount
from 
(
    select 
        userid,
        sigindate,
        row_number() over(partition by userid order by sigindate) as sigin_rank
    from sigin
 ) c 
group by userid, to_date(sigindate,'yyyy-mm-dd')-sigin_rank;
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;得到结果1如下：
```
USERID    DATE_RANK   SIGINCOUNT
1         2017/1/5    3
2         2017/1/6    3
1         2016/12/31  4
2         2016/12/31  3
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上述方法可以找到每个用户的连续活跃天数，但用户中间有中断时程序就无法满足，一个用户出现了多条记录，分别为用户的多段连续活跃所产生。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们最终的目标是得到用户连续活跃的最大天数，可利用上述方法所得到的结果，在外面再嵌套一层，针对userid进行group by，得到每个用户的最大活跃天数。
```
select 
    d.userid, 
    Max(d.sigincount) as max_sigincount 
from (
    select 
        userid,
        to_date(sigindate,'yyyy-mm-dd')-sigin_rank as date_rank,
        count(1) as sigincount
    from (
        select 
            userid,
            sigindate,
            row_number() over(partition by userid order by sigindate) as sigin_rank
        from sigin
    ) c 
    group by userid, to_date(sigindate,'yyyy-mm-dd')-sigin_rank 
) d  
group by d.userid;
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;得到结果2如下：
```
USERID    MAX_SIGINCOUNT
1         4
2         3
```
# 二、如果还需要得到用户连续活跃最大天数中这一段的首次活跃时间，可以把以上两个结果进行关联得到
```
-- 每个用户连续活跃的最大天数和连续活跃的第一天的时间
select 
    f.userid,
    g.date_rank+1,
    f.max_sigincount 
from (
    select 
        d.userid, 
        Max(d.sigincount) as max_sigincount 
    from (
        select 
            userid,
            to_date(sigindate,'yyyy-mm-dd')-sigin_rank as date_rank,
            count(1) as sigincount
        from (
            select 
                userid,
                sigindate,
                row_number() over(partition by userid order by sigindate) as sigin_rank
            from sigin
        ) c 
        group by userid ,to_date(sigindate,'yyyy-mm-dd')-sigin_rank 
    ) d 
    group by d.userid
) f 
inner join 
(
    select 
        userid,
        to_date(sigindate,'yyyy-mm-dd')-sigin_rank  as date_rank,
        count(1) as sigincount
    from (
        select 
            userid,
            sigindate,
            row_number() over(partition by userid order by sigindate) as sigin_rank
        from sigin
    ) c 
    group by userid,to_date(sigindate,'yyyy-mm-dd')-sigin_rank 
) g 
on f.userid = g.userid and f.max_sigincount = g.sigincount;
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;得到结果3如下:
```
USERID    G.DATE_RANK+1   MAX_SIGINCOUNT
2         2017/1/7        3
1         2017/1/1        4
2         2017/1/1        3
```

# 三、问题
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结果3还存在一个问题，如果用户有两段连续活跃的天数相同且最大，则第二段连续活跃的首次活跃时间是不对的，这个问题怎么解决呢？欢迎留言你的解决方案。


- - -
Success often depends upon knowing how long it will take to succeed.