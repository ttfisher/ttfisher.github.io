---
title: Zookeeper菜鸟实录（二）：Paxos算法通俗论
categories:
  - 分布式服务框架之Zookeeper
tags:
  - Zookeeper
comments: true
abbrlink: 8de4cc5d
date: 2018-11-06 09:48:43
---
【引言】基本上来说，作为Java程序员，很少有人会去关心太多关于算法的东西，但作为Zookeeper的核心算法，而且在其他框架中也被多次使用的Paxos，想必还是值得一探究竟的。
<div align=center><img src="http://pm4hdun71.bkt.clouddn.com/img/2018/2018-11-12-02.jpg" width="500"/></div>
<!-- more -->

# 概念
&emsp;&emsp;Paxos算法是莱斯利·兰伯特（Leslie Lamport，就是 LaTeX 中的"La"，此人现在在微软研究院）于1990年提出的一种基于消息传递的一致性算法。这个算法被认为是类似算法中最有效的。
&emsp;&emsp;维基百科是这么解释的：Paxos is a family of protocols for solving consensus in a network of unreliable processors (that is, processors that may fail). Consensus is the process of agreeing on one result among a group of participants. This problem becomes difficult when the participants or their communication medium may experience failures.
&emsp;&emsp;总的来说，Paxos的出现解决的问题是一个分布式系统如何就某个值（决议）达成一致。它允许一组不一定可靠的处理器（服务器）在某些条件得到满足情况下就能达成确定的安全的共识，如果条件不能满足也确保这组处理器（服务器）保持一致。Paxos 算法的目标就是让大家按照少数服从多数的方式，最终达成一致意见。

# 起源
&emsp;&emsp;据说Lamport 是通过故事的方式提出Paxos 问题的，问题的原型是这样的：希腊岛屿Paxon 上的执法者（legislators）在议会大厅（chamber）中表决通过法律，并通过服务员（waiter）传递纸条的方式交流信息，每个执法者会将通过的法律记录在自己的账目（ledger）上。问题在于执法者和服务员都不可靠，他们随时会因为各种事情离开议会大厅，并随时可能有新的执法者进入议会大厅进行法律表决，使用何种方式能够使得这个表决过程正常进行，且通过的法律不发生矛盾？