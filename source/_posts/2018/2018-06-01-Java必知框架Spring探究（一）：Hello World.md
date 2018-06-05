---
title: Java必知框架Spring探究（一）：Hello World
comments: true
categories:
  - Framework Series - Spring
tags:
  - Spring Framework
abbrlink: 8138b98c
date: 2018-06-01 09:09:28
---
【引言】本篇为Spring Framework的开篇之作，作为Java开发人员必知必会的框架，我们的掌握程度到什么火候了呢？恐怕我们很多人也都只是知其然，但却不知其所以然；我一直信奉的一个原则就是：技不贵多，而贵精。所以，不求甚解我是从来不推崇的，这个系列我们就来求一求Spring的甚解。
<div align=center><img src="/img/2018-06-04-01.jpg" width="500"/></div>
<!-- more -->

# 闲话开篇
&emsp;&emsp;事实上，现在大家都在提Spring Boot、Spring Cloud；其实万变不离其宗，归根结底，用到的思想还是Spring那一套；所以这个系列探究，我就从Spring本身说起吧。那么接下来我们来细说一下Spring到底是什么？它有什么用处？为什么要用Spring？
&emsp;&emsp;在细说之前，我们先来集中解释一些概念：
+ Java EE
 + 通常也称为J2EE，实际是是基于这3个概念产生的：J2ME,J2SE,J2EE
 + Java SE=Java Standard Edition，适用于桌面系统的Java平台标准版
 + Java EE=Java Enterprise Edition，适用于创建服务器应用程序和服务的Java平台企业版
 + Java ME=Java Mobile/Micro Edition，适用于小型设备和智能卡
+ POJO
 + POJO（Plain Ordinary Java Object）简单的Java对象，实际就是普通JavaBeans，是为了避免和EJB混淆所创造的简称。
 + POJO的内在含义是指那些没有从任何类继承、也没有实现任何接口，更没有被其它框架侵入的java普通对象。

# Spring的定义
&emsp;&emsp;在wiki上，Spring的定义如下：Spring Framework 是一个开源的Java／Java EE全功能栈（full-stack）的应用程序框架，以Apache License 2.0开源许可协议的形式发布，也有.NET平台上的移植版本。该框架基于 Expert One-on-One Java EE Design and Development（ISBN 0-7645-4385-7）一书中的代码，最初由Rod Johnson和Juergen Hoeller等开发。
&emsp;&emsp;Spring Framework提供了一个简易的开发方式，这种开发方式，将避免那些可能致使底层代码变得繁杂混乱的大量的属性文件和帮助类。Spring是目前最受欢迎的企业级Java应用程序开发框架。粗暴的理解呢，就是Spring是一套比较成熟的企业级Java开发框架，在这个框架上做项目开发很便捷，代码结构也会很清晰，所以现在很多企业和个人都在使用这个框架。框架本身对开发人员来说是一种辅助，更直白的说呢，即使没有任何框架，开发人员也可以把项目做起来，无非就是框架替我们做了很多工作，减轻了我们的开发难度和开发成本。

# Spring的优势

## 低侵入性
&emsp;&emsp;作为一个轻量级的容器框架，Spring是没有侵入性的；通俗的理解：关于高侵入性和低侵入性，比如我们在开发过程中如果自己的代码对框架代码有依赖，那么可定性为高侵入性的（比如：使用某框架时，必须实现它定义的接口或者抽象类，那么实际上框架就侵入了我的代码了）；而低侵入性的框架，在编写自己的代码的时候，不需要考虑框架的任何限制或者必须实现或继承框架的类或者接口，只需要在代码以外添加一些配置即可实现框架的介入（比如：Spring的注解、XML配置等方式实现的依赖注入）

## IoC
&emsp;&emsp;使用IoC容器更加容易组合对象之间的直接关系，面向接口编程，降低耦合；通俗的理解：IoC实现了接口和实现的分离管理，比如当在某个Service中需要使用到一个其他接口的实现，那么我们定义一个接口之后通过注解或者XML即可完成依赖注入，而不需要我们手工去new一个具体的实现，这就很好的降低了类之间的耦合度，做到的面向接口的编程方式（尽量做到定义和实现的分离）。

## AOP
&emsp;&emsp;使用AOP可以更加容易的进行功能扩展；AOP也就是Aspect Oriented Programming的缩写，意为：面向切面编程，也就是通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。通过这个技术，我们可以很容易的对实现的功能进行扩展和缩减而不影响整体。

## Singleton
&emsp;&emsp;Spring创建对象默认是使用了单例模式的，不需要再使用单例模式进行处理；这个就不需要多做解释了吧？如果还不知道什么是单例模式的话，那么我建议咱还是先放下对Spring的研究先去学学设计模式吧。

# Spring的组成
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-04-02.png" width="90%">
&emsp;&emsp;上图是一张经典的Spring框架图，下面针对其中的具体组件简单做一个说明：
+ Spring Core
 + 这是Spring框架最基础的部分，它提供了依赖注入（Dependency Injection）特征来实现容器对Bean的管理。
 + 这里最基本的概念是BeanFactory，它是任何Spring应用的核心。
 + BeanFactory是工厂模式的一个实现，它使用IoC将应用配置和依赖说明从实际的应用代码中分离出来。
+ Spring AOP
 + 全称Aspect Oriented Programming，也就是面向切面编程
 + Spring在它的AOP模块中提供了对面向切面编程的丰富支持。这个模块是在Spring应用中实现切面编程的基础。
 + 为了确保Spring与其它AOP框架的互用性，Spring的AOP支持基于AOP联盟定义的API。
 + AOP联盟是一个开源项目，它的目标是通过定义一组共同的接口和组件来促进AOP的使用以及不同的AOP实现之间的互用性。通过访问他们的站点，你可以找到关于AOP联盟的更多内容。
 + Spring的AOP模块也将元数据编程引入了Spring。使用Spring的元数据支持，你可以为你的源代码增加注释，指示Spring在何处以及如何应用切面函数。
+ Spring Context
 + Spring Core的BeanFactory使Spring成为一个容器，而上下文模块使它成为一个框架。这个模块扩展了BeanFactory的概念，增加了对国际化（I18N）消息、事件传播以及验证的支持。
 + 另外，这个模块提供了许多企业服务，例如电子邮件、JNDI访问、EJB集成、远程以及时序调度（scheduling）服务。也包括了对模版框架例如Velocity和FreeMarker集成的支持。
+ 