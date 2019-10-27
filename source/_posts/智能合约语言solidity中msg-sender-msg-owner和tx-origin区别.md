---
title: '智能合约语言solidity中msg.sender,msg.owner和tx.origin区别'
date: 2019-10-27 16:21:35
tags: [智能合约,msg,sender,owner,origin]
categories: 区块链
notebook: 区块链
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;智能合约solidity语言中自带的变量`msg.sender`, `msg.owner`, `tx.origin`是什么意思呢？来看看。

<img src="智能合约语言solidity中msg-sender-msg-owner和tx-origin区别/solidity.jpeg" width="500" height="300"/>

<!-- more -->

# 1.Difference between msg.owner and msg.sender?
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当部署合约时，msg.sender是合约的所有者，如果合约中定义了一个名为“owner”的变量，则可以为其分配值（地址）msg.sender：
```
address owner = msg.sender;
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此时，变量“owner”将始终具有最初部署合约的人的地址，意味着是合约的所有者。分析这样一行合约代码：owner.transfer(msg.value)，这里如果调用了回退函数（fallback函数），那么msg.value将在owner的地址传输。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个合约的msg.sender是当前与合约交互的地址，可以是用户也可以是另一个合约。所以，如果是一个用户和合约交互，msg.sender是该用户的地址；相反，如果是另一个合约B与该合约交互，msg.sender则是合约B的地址。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;总的来说，msg.sender将会是当前与某合约交互的用户或另一个合约的地址。

# 2.Difference between msg.owner and tx.origin?
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;例如：
```
function setOwner() {
    owner = msg.sender;
}
```
VS
```
function setOwner() {
    owner = tx.origin;
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以这样说，msg.sender的owner可以是一个合约；而tx.origin的owner不可能是一个合约。在一个简单的调用链中，A->B->C->D，D里面的msg.sender将会是C，而tx.origin将一直是A。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是仔细考虑一下是否真的需要使用tx.origin。因为你可能不是合约的唯一用户。其他人可能想要使用你的合约，并希望通过其他的合约与之交互。因为在Solidity中，tx.origin它遍历整个调用栈并返回最初发送调用（或事务）的帐户的地址。在智能合约中使用此变量进行身份验证会使合约容易受到类似网络钓鱼的攻击。

# 3.参考
<b><a>[<font color=#0099ff>Difference between msg.owner and msg.sender and tx.origin</font>](https://ethereum.stackexchange.com/questions/21029/difference-between-msg-owner-and-msg-sender)</b>

- - -
<b>I adopt a dog today, it's great.</b>