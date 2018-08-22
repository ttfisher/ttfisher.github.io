---
title: Java设计模式系列（十五）：解释器模式（Interpreter）
categories:
  - Java one by one - Design Pattern
tags:
  - 设计模式
comments: true
abbrlink: 36c08998
date: 2018-04-16 10:00:00
---
【引言】从各种设计模式的总结历程来看，工厂模式往往是最早提及的，可能是因为这种模式理解起来比较简单，比较直观，所以这里也不例外，将工厂模式作为第一讲吧！
<div align=center><img src="/img/2018/2018-08-20-15.jpg" width="500"/></div>
<!-- more -->

# 言简意赅
&emsp;&emsp;定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。