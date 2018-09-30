---
title: Redis扫盲系列（三）：Redis高级特性（未完成...)
comments: true
categories:
  - Core Technical Series - Message & Cache
tags:
  - Redis
  - Big Data
abbrlink: 99aeaf41
date: 2018-08-02 09:44:00
---
【引言】在掌握了Redis的入门和基本应用之后，我们需要考虑的东西就更偏向于非应用层面的了，比如Redis如何保证高性能，Redis的数据如何存储的，有哪些可用的集群方案，有哪些持久化方案，不同的方案之间的区别和联系都有哪些，和其他MQ的横向对比，等等等等；这些虽然说跟写代码本身关系不大，但是却是关系到整个架构和系统健壮性的，所以，这一章就重点来聊聊这些更高级一些的东西。
<div align=center><img src="/img/2018/2018-08-02-04.jpg" width="500"/></div>
<!-- more -->

# Redis

## 