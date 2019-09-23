---
title: Java必知框架Spring探究（一）：Hello World
comments: true
categories:
  - 技术结构升级
  - Spring全家桶
tags:
  - Spring
abbrlink: 8138b98c
date: 2019-05-06 09:09:28
---
【引言】本篇为Spring Framework的开篇之作，作为Java开发人员必知必会的框架，我们的掌握程度到什么火候了呢？恐怕我们很多人也都只是知其然，但却不知其所以然；我一直信奉的一个原则就是：技不贵多，而贵精。所以，不求甚解我是从来不推崇的，这个系列我们就来求一求Spring的甚解。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-06-04-01.jpg" width="55%"/></div>
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

# Spring的劣势
&emsp;&emsp;生成对象的方式变复杂了（事实上操作还是挺简单的），对于不习惯这种方式的人，会觉得有些别扭和不直观。
&emsp;&emsp;创建对象因为使用了反射技术，在效率上有些损耗。但相对于IoC提高的维护性和灵活性来说，这点损耗是微不足道的，除非某对象的生成对效率要求特别高。

# Spring的组成
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-06-04-02.jpg" width="90%">
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
+ Spring ORM
 + ORM模块对Hibernate、JDO、TopLinkiBatis等ORM框架提供支持
 + ORM模块依赖于dom4j.jar、antlr.jar等包
+ Spring DAO
 + 使用JDBC经常导致大量的重复代码，取得连接、创建语句、处理结果集，然后关闭连接。Spring的JDBC和DAO模块抽取了这些重复代码，因此你可以保持你的数据库访问代码干净简洁，并且可以防止因关闭数据库资源失败而引起的问题。
 + 这个模块还在几种数据库服务器给出的错误消息之上建立了一个有意义的异常层。使你不用再试图破译神秘的私有的SQL错误消息！
 + 另外，这个模块还使用了Spring的AOP模块为Spring应用中的对象提供了事务管理服务。
+ Spring WEB
 + Web上下文模块建立于应用上下文模块之上，提供了一个适合于Web应用的上下文。
 + 这个模块还提供了一些面向服务支持。例如：实现文件上传的multipart请求，它也提供了Spring和其它Web框架的集成，比如Struts、WebWork。
+ Spring WEB MVC
 + Spring为构建Web应用提供了一个功能全面的MVC框架。虽然Spring可以很容易地与其它MVC框架集成，例如Struts，但Spring的MVC框架使用IoC对控制逻辑和业务对象提供了完全的分离。
 + 它也允许你声明性地将请求参数绑定到你的业务对象中，此外，Spring的MVC框架还可以利用Spring的任何其它服务，例如国际化信息与验证。
 + 我们通常开发过程中常说的Spring就是这里的Spring WEB MVC 
 
# SpringBoot

## 什么是spring boot
&emsp;&emsp;Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。用我的话来理解，就是spring boot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像maven整合jar包一样，它整合了很多的框架。Spring 框架就像一个家族，有众多衍生产品例如 boot、security、jpa等等。但他们的基础都是Spring 的 ioc和 aop；ioc 提供了依赖注入的容器 ，aop ，解决了面向横切面的编程，然后在此两者的基础上实现了其他延伸产品的高级功能。于是为了简化开发者的使用，从而创造性地推出了Spring boot，约定优于配置，简化了spring的配置流程。
&emsp;&emsp;说得更简便一些：Spring 最初利用“工厂模式”（DI）和“代理模式”（AOP）解耦应用组件。大家觉得挺好用，于是按照这种模式搞了一个 MVC框架（一些用Spring 解耦的组件），用开发 web 应用（ SpringMVC ）。然后有发现每次开发都写很多样板代码，为了简化工作流程，于是开发出了一些“懒人整合包”（starter），这套就是 Spring Boot。
&emsp;&emsp;spring boot 我理解就是把 spring、 spring mvc 、spring data jpa 等等的一些常用的常用的基础框架组合起来，提供默认的配置，然后提供可插拔的设计，就是各种 starter ，来方便开发者使用这一系列的技术，套用官方的一句话， spring 家族发展到今天，已经很庞大了，作为一个开发者，如果想要使用 spring 家族一系列的技术，需要一个一个的搞配置，然后还有个版本兼容性问题，其实挺麻烦的，偶尔也会有小坑出现，其实蛮影响开发进度， spring boot 就是来解决这个问题，提供了一个解决方案吧，可以先不关心如何配置，可以快速的启动开发，进行业务逻辑编写，各种需要的技术，加入 starter 就配置好了，直接使用，可以说追求开箱即用的效果吧！
&emsp;&emsp;spring大家都知道，boot是启动的意思。所以，spring boot其实就是一个启动spring项目的一个工具而已。从最根本上来讲，Spring Boot就是一些库的集合，它能够被任意项目的构建系统所使用。spring boot并不是一个全新的框架，它不是spring解决方案的一个替代品，而是spring的一个封装。所以，你以前可以用spring做的事情，现在用spring boot都可以做。
&emsp;&emsp;现在流行微服务与分布式系统，springboot就是一个非常好的微服务开发框架，你可以使用它快速的搭建起一个系统。同时，你也可以使用spring cloud（Spring Cloud是一个基于Spring Boot实现的云应用开发工具）来搭建一个分布式的网站。

## 为什么要用spring boot
&emsp;&emsp;其实就是简单、快速、方便！平时如果我们需要搭建一个spring web项目的时候需要怎么做呢？
- 1）配置web.xml，加载spring和spring mvc
- 2）配置数据库连接、配置spring事务
- 3）配置加载配置文件的读取，开启注解
- 4）配置日志文件
- …
- 配置完成之后部署tomcat 调试（首先你还得在本地装一个tomcat）
- …

&emsp;&emsp;如果使用spring boot呢？很简单，我仅仅只需要非常少的几个配置就可以迅速方便的搭建起来一套web项目或者是构建一个微服务！
&emsp;&emsp;spring boot内部整合了很多的第三方组件和框架，比如某一天我们的项目里面突然要用到kafka，那么我们在pom文件这么引入一下，然后就可以使用各种便捷的注解开始我们的开发工作了。
```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

## spring boot优势总结
- 使编码变得简单：spring boot采用java config的方式，对spring进行配置，并且提供了大量的注解，极大地提高了工作效率。
- 使配置变得简单：spring boot提供许多默认配置，当然也提供自定义配置。但是所有spring boot的项目都只有一个配置文件：application.properties/application.yml。用了spring boot，再也不用担心配置出错找不到问题所在了。
- 使部署变得简单：spring boot内置了三种servlet容器：tomcat，jetty，undertow。
- 使监控变得简单：spring boot提供了actuator包，可以使用它来对你的应用进行监控。

## 怎么用spring boot
&emsp;&emsp;这个话题就比较大了，本篇就不做展开了。

# SpringCloud

## 什么是微服务（Micro Service）
&emsp;&emsp;谈到SpringCloud就不得不提微服务的概念，微服务英文名称Microservice，Microservice架构模式就是将整个Web应用组织为一系列小的Web服务；这些小的Web服务可以独立地编译及部署，并通过各自暴露的API接口相互通讯。它们彼此相互协作，作为一个整体为用户提供功能，却可以独立地进行扩展。

## 微服务架构使用场景
- 一般会把整个系统根据业务拆分成几个子系统。
- 每个子系统可以部署多个应用，多个应用之间使用负载均衡。
- 需要一个服务注册中心，所有的服务都在注册中心注册，负载均衡也是通过在注册中心注册的服务来使用一定策略来实现。
- 所有的客户端都通过同一个网关地址访问后台的服务，通过路由配置，网关来判断一个URL请求由哪个服务处理。请求转发到服务上的时候也使用负载均衡。
- 服务之间有时候也需要相互访问。例如有一个用户模块，其他服务在处理一些业务的时候，要获取用户服务的用户数据。
- 需要一个断路器，及时处理服务调用时的超时和错误，防止由于其中一个服务的问题而导致整体系统的瘫痪。
- 还需要一个监控功能，监控每个服务调用花费的时间等。
- 目前主流的微服务框架：Dubbo、 SpringCloud、thrift、Hessian等。

## Spring Cloud是什么
- Spring Cloud是一个微服务框架，相比Dubbo等RPC框架, Spring Cloud提供的全套的分布式系统解决方案。 
- Spring Cloud对微服务基础框架Netflix的多个开源组件进行了封装，同时又实现了和云端平台以及和Spring Boot开发框架的集成。 
- Spring Cloud为微服务架构开发涉及的配置管理，服务治理，熔断机制，智能路由，微代理，控制总线，一次性token，全局一致性锁，leader选举，分布式session，集群状态管理等操作提供了一种简单的开发方式。
- Spring Cloud 为开发者提供了快速构建分布式系统的工具，开发者可以快速的启动服务或构建应用、同时能够快速和云平台资源进行对接。
- 每个Spring Cloud版本，包含着多个不同版本的子项目，为了管理每个版本的子项目清单，避免SpringCloud版本号与其子项目版本号混淆，没有采用版本号方式，而是采用命名方式。这些版本的名字采用了伦敦地铁站的名字，根据字母表顺序来对应版本时间顺序,如：Angel.SR6, Brixton.SR5, Brixton.SR7, Camden.M1.

## Sping Cloud项目结构
&emsp;&emsp;Sping Cloud是Spring的一个顶级项目，Spring的顶级项目列表如下：
- Spring IO platform:用于系统部署，是可集成的，构建现代化应用的版本平台，具体来说当你使用maven dependency引入spring jar包时它就在工作了。
- Spring Boot:旨在简化创建产品级的 Spring 应用和服务，简化了配置文件，使用嵌入式web服务器，含有诸多开箱即用微服务功能，可以和spring cloud联合部署。
- Spring Framework:即通常所说的spring 框架，是一个开源的Java/Java EE全功能栈应用程序框架，其它spring项目如spring boot也依赖于此框架。
- Spring Cloud：微服务工具包，为开发者提供了在分布式系统的配置管理、服务发现、断路器、智能路由、微代理、控制总线等开发工具包。
- Spring XD：是一种运行时环境（服务器软件，非开发框架），组合spring技术，如spring batch、spring boot、spring data，采集大数据并处理。
- Spring Data：是一个数据访问及操作的工具包，封装了很多种数据及数据库的访问相关技术，包括：jdbc、Redis、MongoDB、Neo4j等。
- Spring Batch：批处理框架，或说是批量任务执行管理器，功能包括任务调度、日志记录/跟踪等。
- Spring Security：是一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架。
- Spring Integration：面向企业应用集成（EAI/ESB）的编程框架，支持的通信方式包括HTTP、FTP、TCP/UDP、JMS、RabbitMQ、Email等。
- Spring Social：一组工具包，一组连接社交服务API，如Twitter、Facebook、LinkedIn、GitHub等，有几十个。
- Spring AMQP：消息队列操作的工具包，主要是封装了RabbitMQ的操作。
- Spring HATEOAS：是一个用于支持实现超文本驱动的 REST Web 服务的开发库。
- Spring Mobile：是Spring MVC的扩展，用来简化手机上的Web应用开发。
- Spring for Android：是Spring框架的一个扩展，其主要目的在乎简化Android本地应用的开发，提供RestTemplate来访问Rest服务。
- Spring Web Flow：目标是成为管理Web应用页面流程的最佳方案，将页面跳转流程单独管理，并可配置。
- Spring LDAP：是一个用于操作LDAP的Java工具包，基于Spring的JdbcTemplate模式，简化LDAP访问。
- Spring Session：session管理的开发工具包，让你可以把session保存到redis等，进行集群化session管理。
- Spring Web Services：是基于Spring的Web服务框架，提供SOAP服务开发，允许通过多种方式创建Web服务。
- Spring Shell：提供交互式的Shell可让你使用简单的基于Spring的编程模型来开发命令，比如Spring Roo命令。
- Spring Roo：是一种Spring开发的辅助工具，使用命令行操作来生成自动化项目，操作非常类似于Rails。
- Spring Scala：为Scala语言编程提供的spring框架的封装（新的编程语言，Java平台的Scala于2003年底/2004年初发布）。
- Spring BlazeDS Integration：一个开发RIA工具包，可以集成Adobe Flex、BlazeDS、Spring以及Java技术创建RIA。
- Spring Loaded：用于实现java程序和web应用的热部署的开源工具。
- Spring REST Shell：可以调用Rest服务的命令行工具，敲命令行操作Rest服务。

## Spring Cloud的子项目
- Spring Cloud Config：配置管理工具，支持使用Git存储配置内容，支持应用配置的外部化存储，支持客户端配置信息刷新、加解密配置内容等
- Spring Cloud Bus：事件、消息总线，用于在集群（例如，配置变化事件）中传播状态变化，可与Spring Cloud Config联合实现热部署。
- Spring Cloud Netflix：针对多种Netflix组件提供的开发工具包，其中包括Eureka、Hystrix、Zuul、Archaius等。
 - Netflix Eureka：一个基于rest服务的服务治理组件，包括服务注册中心、服务注册与服务发现机制的实现，实现了云端负载均衡    和中间层服务器的故障转移。
 - Netflix Hystrix：容错管理工具，实现断路器模式，通过控制服务的节点,从而对延迟和故障提供更强大的容错能力。
 - Netflix Ribbon：客户端负载均衡的服务调用组件。
 - Netflix Feign：基于Ribbon和Hystrix的声明式服务调用组件。
 - Netflix Zuul：微服务网关，提供动态路由，访问过滤等服务。
 - Netflix Archaius：配置管理API，包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、轮询框架、回调机制等功能。
- Spring Cloud for Cloud Foundry：通过Oauth2协议绑定服务到CloudFoundry，CloudFoundry是VMware推出的开源PaaS云平台。
- Spring Cloud Sleuth：日志收集工具包，封装了Dapper,Zipkin和HTrace操作。
- Spring Cloud Data Flow：大数据操作工具，通过命令行方式操作数据流。
- Spring Cloud Security：安全工具包，为你的应用程序添加安全控制，主要是指OAuth2。
- Spring Cloud Consul：封装了Consul操作，consul是一个服务发现与配置工具，与Docker容器可以无缝集成。
- Spring Cloud Zookeeper：操作Zookeeper的工具包，用于使用zookeeper方式的服务注册和发现。
- Spring Cloud Stream：数据流操作开发包，封装了与Redis,Rabbit、Kafka等发送接收消息。
- Spring Cloud CLI：基于 Spring Boot CLI，可以让你以命令行方式快速建立云组件。
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-08-27-01.jpg" width="75%">