---
title: Java必知框架Spring探究（二）：细说SpringMVC
comments: true
categories:
  - Core Technical Series - Spring Plus
tags:
  - Spring
abbrlink: 1ace9dad
date: 2018-06-07 09:09:28
---
【引言】谈到web开发，可能大多数Java人的第一印象就是SpringMVC，它是我们最熟悉的一个框架，但系统的来说能把SpringMVC说透的人不多，所以我这里也尝试一下来自己理一理SpringMVC这个既熟悉又陌生的框架。
<div align=center><img src="/img/2018/2018-08-27-02.jpg" width="500"/></div>
<!-- more -->

# 闲话开篇
&emsp;&emsp;本章不打算细到代码层面的去深究（当然后面如果时间允许，是可以花一段时间解读一下SpringMVC的源码的，同时可以稍微花点时间研究研究官方手册，权当学英语了），所以本章我们从大的方面来解决以下几个问题：
- SpringMVC是什么？
- SpringMVC的运行流程是怎么样子的？
- SpringMVC的一些常用特性有哪些
- 最后，用SpringMVC写个最最简单的Demo

# SpringMVC是什么？
&emsp;&emsp;首先提一下，百度了一下SpringMVC，通过spring.io进入的官网已经被Spring Boot和Cloud霸屏了，时代总是在不断向前发展的，技术也不例外。
&emsp;&emsp;官方解释如下：Spring Web MVC is the original web framework built on the Servlet API and included in the Spring Framework from the very beginning. The formal name "Spring Web MVC" comes from the name of its source module spring-webmvc but it is more commonly known as "Spring MVC".看来官方的描述也透露着一股古老的气息，very beginning......
&emsp;&emsp;MVC=Model-View-Controller，一种软件设计思想，将软件分为三层：模型层（具体的业务处理）、视图层（用户交互）、控制层（请求分发）。
&emsp;&emsp;Spring MVC是Spring对MVC思想的一种实现，建立在Spring核心功能之上，功能强大，使用方便。

# SpringMVC的运行流程

<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-08-27-03.jpg" width="75%">

## 概览
&emsp;&emsp;上图是说SpringMVC流程的一张经典图例，基本上整个流程该图都涵盖了：
- 用户发送请求到前端控制器（DispatcherServlet）。
- 前端控制器请求处理器映射器（HandlerMapping）去查找处理器（Handler）。
- 找到以后处理器映射器（HandlerMappering）向前端控制器返回执行链（HandlerExecutionChain）。
- 前端控制器（DispatcherServlet）调用处理器适配器（HandlerAdapter）去执行处理器（Handler）。
- 处理器适配器去执行Handler。
- 处理器执行完给处理器适配器返回ModelAndView。
- 处理器适配器向前端控制器返回ModelAndView。
- 前端控制器请求视图解析器（ViewResolver）去进行视图解析。
- 视图解析器向前端控制器返回View。
- 前端控制器对视图进行渲染。
- 前端控制器向用户响应结果。

## 核心组件
&emsp;&emsp;SpringMVC的核心组件清单如下，以下是简单的一个说明，后面会针对每个组件展开详细说明。
- 前端控制器（DisatcherServlet）:接收请求，响应结果，返回可以是json,String等数据类型，也可以是页面（Model）。
- 处理器映射器（HandlerMapping）:根据URL去查找处理器，一般通过xml配置或者注解进行查找。
- 处理器（Handler）：就是我们常说的controller控制器啦，由程序员编写。
- 处理器适配器（HandlerAdapter）:可以将处理器包装成适配器，这样就可以支持多种类型的处理器。
- 视图解析器（ViewResovler）:进行视图解析，返回view对象（常见的有JSP,FreeMark等）。

## DisatcherServlet

### Servlet
&emsp;&emsp;DispatcherServlet的本质也是一个Servlet，所以要理解DispatcherServlet，首先要知道Servlet是什么。
&emsp;&emsp;Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。使用 Servlet，您可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录，还可以动态创建网页。Java Servlet 通常情况下与使用 CGI（Common Gateway Interface，公共网关接口）实现的程序可以达到异曲同工的效果。但是相比于 CGI，Servlet 在性能上，通用性上，资源安全性上，独立性上都是占有优势的。
&emsp;&emsp;借用Tomcat Server的结构简图，可以简单理解Servlet是什么了：
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-08-27-04.jpg" width="75%">

### 基本配置
&emsp;&emsp;在传统的SpringMVC模式的开发过程中，web.xml是一个比较核心的东西，所以理所当然的DispatcherServlet也是在这里配置的。
```xml
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>*.json</url-pattern>
</servlet-mapping>
```

### 几个参数
&emsp;&emsp;contextConfigLocation：DispatcherServlet默认从WEB-INF目录下加载SpringMVC的配置文件，可以通过属性contextConfigLocation更改配置文件的位置。
&emsp;&emsp;load-on-startup：默认情况下，Servlet在被请求时才实例化初始化，如果希望在服务器启动时创建Servlet对象，可以通过<load-on-startup>标签设定。
&emsp;&emsp;url-pattern：url转发的规则；一般写成*加后缀的形式，如“*.do”，不能写成这种形式“/*”，这种形式会把Web服务器接收到的请求全部转发给DispatcherServlet，而SpringMVC容器中不一定存在对应的Handler，会因找不到对应的资源而报错，比如无法加载HTML、JSP页面。

## HandlerMapping

### 用途
&emsp;&emsp;访问资源时使用的不是资源的全限定性类名或者其他可以直接确定资源的方式，而是采用url，这样就需要在访问方式与与资源之间建立起一对一的关系，这种关系就是映射关系，HandlerMapping就负责创建与解析这种关系，根据访问方式确定处理器。

### 类别
- BeanNameUrlHandlerMapping：默认的处理器映射器，url与beanName相同。
- DefaultAnnotationHandlerMapping：注解开发时默认的处理器映射器。
- SimpleUrlHandlerMapping：自定义url，在url与beanName之间建立映射关系：