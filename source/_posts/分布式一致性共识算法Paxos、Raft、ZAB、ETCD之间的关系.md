---
title: 分布式一致性共识算法Paxos、Raft、ZAB、ETCD之间的关系
date: 2019-10-01 11:41:21
tags: [分布式,共识算法,Paxos,Raft,Zab,Etcd]
categories: 共识算法
notebook: 区块链
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在分布式系统中，多个节点之间达成共识成为最重要的组成部分，在发展过程中，出现许多类型的算法，他们从最基本的达成共识，到易于理解，更接近于实践应用等等方面，做到了极致，下面来看看Paxos、Raft、ZAB、Etcd之间的不同。

![consensus_algorithm_difference](分布式一致性共识算法Paxos、Raft、ZAB、ETCD之间的关系/consensus_algorithm_difference.jpeg)

<!-- more -->

1. 为了解决分布式系统中的一致性问题，科学家们首先提出了Paxos算法，但是Paxos流程太过繁杂，不易于理解，应用主要有Chubby、libpaxos；
2. 斯坦福大学的2个人以易于理解为目标，又能实现Paxos所解决的问题，于是实现了Raft算法，到现在已经有了十多种语言的Raft算法实现框架，较为出名的有etcd，Google的Kubernetes也是用了etcd作为他的服务发现框架；
3. Zab与Paxos不同，但是有些地方是从Paxos那里学习过来的，比如Leader发送心跳、议案、决定等给Follower；Leader在提交（commit）议案之前需要经过一定量的Follower的确认；算法实现最经典的应用为zookeeper；
4. 但Etcd和Zab不适合分布式大数据存储，主要做分布式元数据的存储；


- - -
when your ability don't support your ambition, then you should learn down-to-earth.
