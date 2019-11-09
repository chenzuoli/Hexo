---
title: Presto-SQL与Hive-SQL的区别与联系
date: 2019-11-09 23:45:51
tags: [Presto,OLAP,Hive,SQL]
categories: 数据仓库
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Presto作为一个数据仓库后端的OLAP分析查询系统，提供了比Hive更快的查询速度，能及时提供用户的分析结果数据，深受OLAP用户喜爱，那么我们从hive迁移到presto时他们的查询方式有哪些区别与联系呢，下面来看看。

<img src="Presto-SQL与Hive-SQL的区别与联系/presto.jpeg" width="500" height="300"/>

<!-- more -->

# 一、前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Presto使用ANSI SQL语法和语义，而Hive使用类似SQL的语言，称为HiveQL，它在MySQL（它本身与ANSI SQL有很多不同）之后进行了松散的建模。

# 二、使用下标来访问数组的动态索引而不是udf
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SQL中的下标运算符支持完整表达式，与Hive（仅支持常量）不同。因此，您可以编写如下查询：
```
SELECT my_array[CARDINALITY(my_array)] as last_element
FROM ...
```

# 三、避免超出阵列的边界访问
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;访问数组的超出边界元素将导致异常。您可以通过以下方式避免这种if情况：
```
SELECT IF(CARDINALITY(my_array) >= 3, my_array[3], NULL)
FROM ...
```
# 四、对数组使用ANSI SQL语法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;数组从1开始索引，而不是从0开始：
```
SELECT my_array[1] AS first_element
FROM ...
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用ANSI语法构造数组：
```
SELECT ARRAY[1, 2, 3] AS my_array
```

# 五、对标识符和字符串使用ANSI SQL语法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字符串用单引号分隔，标识符引用双引号，而不是反引号：
```
SELECT name AS "User Name"
FROM "7day_active"
WHERE name = 'foo'
```

# 六、引用以数字开头的标识符
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以数字开头的标识符在ANSI SQL中不合法，必须使用双引号引用：
```
SELECT *
FROM "7day_active"
```

# 七、使用标准字符串连接运算符
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用ANSI SQL字符串连接运算符：
```
SELECT a || b || c
FROM ...
```

# 八、使用CAST目标的标准类型
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CAST目标支持以下标准类型：
```
SELECT
  CAST(x AS varchar)
, CAST(x AS bigint)
, CAST(x AS double)
, CAST(x AS boolean)
FROM ...
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;特别是，使用VARCHAR而不是STRING。

# 九、除以整数时使用CAST
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Presto遵循分割两个整数时执行整数除法的标准行为。例如，除以7由2将导致3，而不是3.5。要对两个整数执行浮点除法，请将其中一个转换为double：
```
SELECT CAST(5 AS DOUBLE) / 2
```

# 十、使用WITH表示复杂的表达式或查询
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果要将复杂输出表达式重用为过滤器，请使用内联子查询或使用WITH子句将其分解：
```
WITH a AS (
  SELECT substr(name, 1, 3) x
  FROM ...
)
SELECT *
FROM a
WHERE x = 'foo'
```

# 十一、使用UNNEST扩展数组和映射
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Presto支持UNNEST扩展阵列和地图。用UNNEST而不是。LATERAL VIEW explode()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Hive查询：
```
SELECT student, score
FROM tests
LATERAL VIEW explode(scores) t AS score;
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Presto查询：
```
SELECT student, score
FROM tests
CROSS JOIN UNNEST(scores) AS t (score);
```

- - -
<b>If I were you, I would do what I want.</b>