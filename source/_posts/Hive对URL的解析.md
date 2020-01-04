---
title: Hive对URL的解析
date: 2020-01-03 19:01:37
tags: [Hive,URL]
categories: 数据仓库
notebook: 数据仓库
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有时，业务库直接将页面的url存储在日志表中，访问的page携带参数，此时获取参数就成为了问题，需要从字段中截取，那么hive sql如何解析url呢，来看看。

<img src="Hive对URL的解析/sky.jpeg" width="500" height="300"/>

<!-- more -->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Every HTTP URL conforms to the syntax of a generic URI. A generic URI is of the form:

```
scheme:[//[user:password@]host[:port]][/]path[?query][#fragment]
```

# 1.从url返回PROTOCOL
```
hive> select parse_url('https://www.baidu.com/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'PROTOCOL');
OK
https
```

# 2.从url返回HOST
```
hive> select parse_url('https://www.baidu.com/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'HOST');
OK
www.baidu.com
```

# 3.从url返回PATH
```
hive> select parse_url('https://www.baidu.com/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'PATH');
OK
/s
```

# 4.从url返回QUERY
```
hive> select parse_url('https://www.baidu.com/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'QUERY');
OK
cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage
```

# 5.从url返回QUERY中指定的参数的值
```
hive> select parse_url('https://www.baidu.com/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'QUERY', 'wd');
OK
大唐不夜城着火
```

# 6.从url返回FRAGMENT标识符
```
hive> select parse_url('https://www.baidu.com/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage#REGEXP_REPLACT', 'REF');
OK
REGEXP_REPLACT
```

# 7.从url返回FILE
```
//一般为 path?query
hive> select parse_url('https://www.baidu.com/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'FILE');
OK
/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage
 
hive> select parse_url('https://tester:123456@www.baidu.com:8888/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'FILE');
OK
/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage
 
hive> select parse_url('https://tester:123456@www.baidu.com:8888/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage#REGEXP_REPLACT', 'FILE');
OK
/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage
```

# 8.从url返回AUTHORITY
```
//userinfo@host:port
hive> select parse_url('https://www.baidu.com/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'AUTHORITY');
OK
www.baidu.com
 
hive> select parse_url('https://www.baidu.com:8888/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'AUTHORITY');
OK
www.baidu.com:8888
 
hive> select parse_url('https://tester:123456@www.baidu.com:8888/s?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'AUTHORITY');
OK
tester:123456@www.baidu.com:8888
```

# 9.从url返回USERINFO
```
hive> select parse_url('http://tester:123456@www.baidu.com?cl=3&tn=baidutop10&fr=top1000&wd=大唐不夜城着火&rsv_idx=2&rsv_dl=fyb_n_homepage', 'USERINFO');
OK
tester:123456
```

- - -
<b>It's never too late to learn.</b>