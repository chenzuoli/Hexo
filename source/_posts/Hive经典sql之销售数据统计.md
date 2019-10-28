---
title: Hive经典sql之销售数据统计
date: 2019-10-28 11:01:51
tags: [Hive, SQL]
categories: 数据仓库
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;现在有这样一个需求：一张表`sales`中记录了公司每个业务销售的每月销售数据，来分析下2个案例。

<img src="Hive经典sql之销售数据统计/time_fly.jpeg" width="500" height="300"/>

<!-- more -->

# 1.sales表结构
```
| month  | user_id | amount |
+--------+---------+--------+
| 201601 | 1       |    500 |
| 201601 | 2       |    300 |
| 201601 | 3       |    100 |
| 201602 | 1       |   1000 |
| 201602 | 2       |    800 |
| 201603 | 2       |   1000 |
| 201603 | 3       |    500 |
| 201604 | 1       |   1000 |
```

# 2.需求一：求每个月的销售额及累计销售额
开窗函数实现：
```
SELECT 
    month,
    SUM(amount) AS cur_month_amount,
    SUM(SUM(amount)) OVER (ORDER BY month) 
FROM sales 
GROUP BY month ORDER BY month;
```
或者
```
SELECT
    month,
    SUM(amount) OVER(PARTITION BY month) AS cur_month_amount,
    SUM(amount) OVER(ORDER BY month) AS sum_amount
FROM sales;
```

# 3.需求二：每个月的销售冠军和销售额
开窗函数实现：
```
SELECT
    month,
    user_id,
    MAX(SUM(amount)) OVER(PARTITION BY month, user_id) AS cur_month_top_amount
FROM sales;
```
或者
```
SELECT
    month,
    user_id,
    amount
FROM (
    SELECT
        month,
        user_id,
        amount,
        ROW_NUMBER() OVER(PARTITION BY month ORDER BY amount DESC) AS rank
    FROM sales
) a
where rank = 1;
```

- - -
<b>How time flies.</b>