---
title: Hexo添加置顶功能
date: 2019-12-08 22:47:08
tags: [Hexo,置顶]
categories: Hexo
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;目前，Hexo项目默认按照创建日期进行的排序，置顶功能如何实现呢，下面来看看吧。

<img src="Hexo添加置顶功能/leaf.jpeg" width="500" height="300"/>

<!-- more -->

1. 修改node_modules/hexo-generator-index/lib/generator.js
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;原理就是，先按照设定的top值进行排序，再按照日期排序：
```
'use strict';
var pagination = require('hexo-pagination');
module.exports = function(locals){
  var config = this.config;
  var posts = locals.posts;
    posts.data = posts.data.sort(function(a, b) {
        if(a.top && b.top) { // 两篇文章top都有定义
            if(a.top == b.top) return b.date - a.date; // 若top值一样则按照文章日期降序排
            else return b.top - a.top; // 否则按照top值降序排
        }
        else if(a.top && !b.top) { // 以下是只有一篇文章top有定义，那么将有top的排在前面（这里用异或操作居然不行233）
            return -1;
        }
        else if(!a.top && b.top) {
            return 1;
        }
        else return b.date - a.date; // 都没定义按照文章日期降序排
    });
  var paginationDir = config.pagination_dir || 'page';
  return pagination('', posts, {
    perPage: config.index_generator.per_page,
    layout: ['index', 'archive'],
    format: paginationDir + '/%d/',
    data: {
      __index: true
    }
  });
};
```

2. 设置文章置顶
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在文章Front-matter中添加top值，数值越大文章越靠前，如：
```
---
title: Hexo+nexT主题配置备忘
date: 2019-12-08 11:49:33
tags: [Hexo,next-theme,Seo]
categories: Hexo
top: 10
---
```

- - -
<b>When you are talking about something in front of so many people, don't forget to smile.</b>