---
title: Solidity中transfer和send的区别
date: 2019-10-28 20:32:43
tags: [Solidity, transfer, send]
categories: 区块链
notebook: 区块链
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;智能合约中我们可以根据不同的场景来使用这2中方式。下面来看看解释。

<img src="Solidity中transfer和send的区别/send.jpeg" width="500" height="300"/>

<!-- more -->

<b>1. address.transfer()</b>
- throws on failure     // 转账失败会抛出异常
- forwards 2,300 gas stipend (not adjustable), safe against reentrancy
- should be used in most cases as it's the safest way to send ether


<b>2. address.send()</b>
- returns false on failure  // 转账失败会返回false
- forwards 2,300 gas stipend (not adjustable), safe against reentrancy
- should be used in rare cases when you want to handle failure in the contract


<b>3. address.call.value().gas()()</b>
- returns false on failure
- forwards all available gas (adjustable), not safe against reentrancy
- should be used when you need to control how much gas to forward when sending ether or to call a function of another contract

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;函数返回值就是transfer和send的最主要区别，生产中用transfer比较多。

- - -
<b>Where there is a life, there is a hope.</b>