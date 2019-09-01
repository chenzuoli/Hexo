---
title: go语言学习
date: 2019-08-10 09:06:33
tags: go
categories: 计算机语言
notebook: Dapp&Smart Contract Develop
---

go语言: 面向对象、强类型、类似c的语言。
![golang](go语言学习/golanguage.png)
<!-- more -->

> 1.同一个目录下面不能有多个package main，分到不同的文件夹中即可；
> 2.go test
>> 1. *_test.go是golang特有的约定，为测试文件: go run: cannot run *_test.go files;
>> 2. go test 默认执行当前目录下以xxx_test.go的测试文件;
>> 3. go test -v 可以看到详细的输出信息;
>> 4. go test -v xxx_test.go 指定测试单个文件，但是该文件中如果调用了其它文件中的模块会报错;
>> 5. go test -v -test.run Testxxx, 该测试会测试包含该函数名的所有函数.
> 3.函数修饰符view：只能读取数据，不能更改数据；修饰符pure：不访问程序中的数据，他的返回值完全取决于传入的参数

测试代码见<a>[github](https://github.com/chenzuoli/learn-go-with-tests)</a>

2019-08-25更新
————————————
当你累了的时候，停下来做个梦吧。
愿你坚持到底。

2019-09-01
————————————
推荐给大家一个非常好的入门学习中文网站，里面很全，从基本数据类型、语法，到协程并发、高阶函数、类、多态等。
<a>[go语言中文网](https://studygolang.com/subject/2)</a>

# panic和recover
参考文档：<a>[panic和recover](https://studygolang.com/articles/12785)</a>

# 头等函数
参考文档：<a>[头等函数](https://studygolang.com/articles/12789)</a>

# 反射
参考文档：<a>[反射](https://studygolang.com/articles/13178)</a>

# 读取文件
参考文档：<a>[读取文件](https://studygolang.com/articles/14669)</a>

# 写入文件/并发写入
参考文档：<a>[写入文件/并发写入](https://studygolang.com/articles/19475)</a>


当你累了的时候，停下来做个梦吧。
愿你坚持到底。