---
title: Java必知框架MyBatis探究（二）：跟着Demo看源码（未完成...）
comments: true
categories:
  - 经典SSM架构之MyBatis
tags:
  - MyBatis
abbrlink: 2f64b75a
date: 2018-08-30 08:29:28
---
【引言】前一章做了个简单的Demo程序，实现了最基本的CRUD操作，但我们的诉求并不止于会用MyBatis完成CRUD，所以本章我们就着这个Demo来了解一下MyBatis内部的流程，顺便看一看框架的源码！
<div align=center><img src="http://pm4hdun71.bkt.clouddn.com/img/public/000023.jpg" width="500"/></div>
<!-- more -->

# 闲言
&emsp;&emsp;像这种不是特别庞大的工具型框架，最好的学习就是通过Demo来熟悉，所以这次研究MyBatis的第一站，我打算从时下相当流行的springboot来结合着通过一个小Demo先了解一下，入个门。