---
title: 一致性共识算法Raft图文详解
date: 2019-09-29 14:10:11
tags: [一致性协议,raft,共识算法]
categories: [区块链,共识算法]
notebook: 区块链
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;熟悉或了解分布式系统的开发者都知道一致性算法的重要性，Paxos一致性算法提出至今已经有二十几年了，而Paxos流程太过于繁杂，实现起来比较复杂，因此Raft应运而生，它是比Paxos更简单而又能实现Paxos所解决的问题的一致性算法。

![raft](一致性共识算法图文详解/raft.jpeg)

<!-- more -->

</br>

<a>[<font color=#0099ff><b>Raft动图展示详解</b></font>](http://thesecretlivesofdata.com/raft/?spm=a2c4e.10696291.0.0.122c19a4sBpxKb)</a>

- - -
Death is like the wind, always by my side.