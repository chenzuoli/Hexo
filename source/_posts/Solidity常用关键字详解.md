---
title: Solidity常用关键字详解
date: 2019-10-27 16:46:43
tags: [智能合约,solidity,关键字]
categories: 区块链
notebook: 区块链
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最近在学习公链中的智能合约语言Solidity，下面来看看自带关键字的一些解释吧。

<img src="Solidity常用关键字详解/solidity_keyword.jpeg" width="500" height="300"/>

<!-- more -->

# 1.require
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;若require函数的第一个参数执行结果为false，则终止执行，撤销所有对状态和以太坊余额的改动，在旧版的EVM中会消耗所有gas，但现在不会了，你也可以在函数的第二个参数中对错误进行解释。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用require来检查函数是否被正确调用，是一个好习惯。
```
require(
    msg.sender == chairman,
    "Only chairman can give the right to vote."
);
```

# 2.payable
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;函数修饰符：payable关键字，如果一个函数需要进行货币操作，必须要带上payable关键字，这样才能正常接收msg.value。


# 3.msg.sender/msg.owner/tx.origin
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考下这篇文章：<b><a>[<font color=#0099ff>Difference between msg.owner and msg.sender and tx.origin</font>](http://wetech.top/2019/10/27/%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%E8%AF%AD%E8%A8%80solidity%E4%B8%ADmsg-sender-msg-owner%E5%92%8Ctx-origin%E5%8C%BA%E5%88%AB/)</b>


# 4.msg.value
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;默认为给合约转账的金额。

# 5.this.balance
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当前合约中的余额。



- - -
<b>The best way to escape from your problem is to solve it.</b>