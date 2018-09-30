---
title: Java必知框架MyBatis探究（一）：Springboot版Demo（未完成...）
comments: true
categories:
  - Core Technical Series - JPA Plus
tags:
  - MyBatis
abbrlink: '85554181'
date: 2018-08-29 08:29:28
---
【引言】记得前段时间参加阿里的电话面试，被面试官问到MyBatis的一级缓存和二级缓存的知识，结果自然是No Answer；所以便考虑着抽些时间来看看MyBatis的源码吧，一来让自己更熟悉一些别人是怎么做这种组件实现的，二来到了现在这个阶段，该往深处看看了！
<div align=center><img src="/img/2018/2018-08-30-01.jpg" width="500"/></div>
<!-- more -->

# 闲言
&emsp;&emsp;像这种不是特别庞大的工具型框架，最好的学习就是通过Demo来熟悉，所以这次研究MyBatis的第一站，我打算从时下相当流行的springboot来结合着通过一个小Demo先了解一下，入个门。