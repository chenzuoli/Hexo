---
title: go语言学习
date: 2019-08-10 09:06:33
tags: go
categories: 计算机语言
notebook: Dapp&Smart Contract Develop
---

go语言: 面向对象、强类型、类似c的语言。

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