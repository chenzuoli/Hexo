---
title: Hive经典sql之花费当月费用与总费用
date: 2019-10-28 10:24:41
tags: [Hive, SQL]
categories: 数据仓库
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;现在有这样一个需求：一张表`orders`中记录了每个用户消费的明细（时间、金额），现在需要求出每个用户当月的消费总金额和总的消费总金额。如何求呢？

<img src="Hive经典sql之花费当月费用与总费用/hidi_and_grandpa.jpeg" width="500" height="300"/>

<!-- more -->

# 1.orders表结构
```
name    string
dt      date
amount  double
```

# 2.分析
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;统计的当月总金额这个汇总值，那么对`name`和`month`进行分组`sum(amount)`即可得到。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;统计总金额，那么只需对`name`进行分组即可。
1. 方法一
分别求出上面2个汇总值，然后进行join就行：
```
select
    name,
    month_amount,
    sum_amount
from (
    select
        name,
        month(dt) as cur_month,
        sum(amount) as month_amount
    from orders
    group by name, month(dt);
) t1
join
(
    select
        name,
        sum(amount) as sum_amount
    from orders
    group by name
) t2
on t1.name = t2.name;
```

2. 方法二
使用开窗函数，不在需要表join了：
```
select
    name,
    sum(amount) over(partition by name, month(dt)) as month_amount,
    sum(amount) over(partition by name) as sum_amount
from orders
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开窗函数是不是简单多了？

- - -
<b>If somethings in life brings you joy, then you simple have to do it, regardless of what people say.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---《海蒂与爷爷》</b>