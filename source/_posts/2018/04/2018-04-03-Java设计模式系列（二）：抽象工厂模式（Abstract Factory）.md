---
title: Java设计模式系列（二）：抽象工厂模式（Abstract Factory）
categories:
  - 【103】衣带渐宽终不悔之我看设计模式
tags:
  - 设计模式
comments: true
abbrlink: a549a21f
date: 2018-04-03 10:00:00
---
【引言】从工厂到抽象工厂，两个其实容易混淆，所以本章打算好好对比以下工厂模式系列！
<div align=center><img src="/img/2018/2018-08-20-02.jpg" width="500"/></div>
<!-- more -->

# 工厂模式Plus

## 简单工厂模式
&emsp;&emsp;工厂类中，根据条件决定一个接口由哪个具体产品类来实现。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-08-28-01.jpg" width="75%">

## 工厂模式
&emsp;&emsp;创建多个工厂类。各个工厂类中，都对应一个获得接口A实例的方法。用户决定使用哪个工厂。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-08-28-02.jpg" width="75%">

## 抽象工厂模式
&emsp;&emsp;对工厂方法进行扩展。各个工厂类中，再增加一个获得接口B实例的方法。抽象工厂和工厂方法没有本质区别，是对工厂方法的扩展。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-08-28-03.jpg" width="75%">


# 基本特性

## 一句话介绍
&emsp;&emsp;定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。