---
title: OpenResty应用系列（一）：基础概念认知
categories:
  - 分布式服务辅助之OpenResty
tags:
  - OpenResty
abbrlink: 4fbd6846
date: 2018-09-11 14:26:00
---
【引言】对于大部分的web应用，出于性能和扩展性以及安全性考虑，都会或多或少的走上负载均衡反向代理这条路，反向代理组件中最常见的也就是Nginx了，而OpenResty在Nginx基础上有了更多的赋能（比如支持Lua），所以在很多场景下OpenResty可以帮我们很简单的完成一些业务不便完成的东西，这个系列就来讨论讨论从Nginx到OpenResty的前世今生。
<div align=center><img src="/img/2018/2018-09-12-09.jpg" width="500"/></div>
<!-- more -->

# OpenResty是什么？
&emsp;&emsp;OpenResty® (又称：ngx_openresty) ；是一个基于 Nginx 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。
&emsp;&emsp;OpenResty® 通过汇聚各种设计精良的 Nginx 模块（主要由 OpenResty 团队自主开发），从而将 Nginx 有效地变成一个强大的通用 Web 应用平台。这样，Web 开发人员和系统工程师可以使用 Lua 脚本语言调动 Nginx 支持的各种 C 以及 Lua 模块，快速构造出足以胜任 10K 乃至 1000K 以上单机并发连接的高性能 Web 应用系统。
&emsp;&emsp;OpenResty® 的目标是让你的Web服务直接跑在 Nginx 服务内部，充分利用 Nginx 的非阻塞 I/O 模型，不仅仅对 HTTP 客户端请求,甚至于对远程后端诸如 MySQL、PostgreSQL、Memcached 以及 Redis 等都进行一致的高性能响应。
&emsp;&emsp;通俗的理解：OpenResty就是以Nginx为核心，包了一个Lua支持的外壳；就是支持Lua语言扩展的Nginx升级版。

# Nginx是什么？
&emsp;&emsp;Nginx（发音同engine x）是一个异步框架的 Web服务器，也可以用作反向代理，负载平衡器 和 HTTP缓存。该软件由 Igor Sysoev 创建，并于2004年首次公开发布。同名公司成立于2011年，以提供支持。
Nginx是一款免费的开源软件，根据类BSD许可证的条款发布。一大部分Web服务器使用Nginx，通常作为负载均衡器。其特点是占有内存少，并发能力强。从网上看到个还不错的参考图：
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-01.jpg" width="75%">

# 为什么选择Nginx？
+ 处理响应请求很快
+ 高并发连接，Nginx支持的并发连接上限取决于你的内存
+ 低的内存消耗，在一般的情况下，10000个非活跃的HTTP Keep-Alive 连接在Nginx中仅消耗2.5MB的内存，这也是Nginx支持高并发连接的基础。
+ 具有很高的可靠性、高扩展性、支持热部署
+ 自由的BSD许可协议，BSD许可协议不只是允许用户免费使用Nginx，也允许用户修改Nginx源码，还允许用户用于商业用途。

# Lua是什么？
&emsp;&emsp;Lua 是一个小巧的脚本语言。是巴西里约热内卢天主教大学（Pontifical Catholic University of Rio de Janeiro）里的一个研究小组，由Roberto Ierusalimschy、Waldemar Celes 和 Luiz Henrique de Figueiredo所组成并于1993年开发。其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。Lua由标准C编写而成，几乎在所有操作系统和平台上都可以编译，运行。Lua并没有提供强大的库，这是由它的定位决定的。所以Lua不适合作为开发独立应用程序的语言。Lua 有一个同时进行的JIT项目，提供在特定平台上的即时编译功能。

# Nginx请求处理的流程参考
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-02.jpg" width="75%">

# 如何理解反向代理？

## 什么是反向代理？
&emsp;&emsp;所谓的代理，可以理解为一个中介，它屏蔽用户和服务提供者之间的直接接触，比如A和B本来可以直连，中间插入一个C，C就是中介。刚开始的时候，代理多数是帮助内网client访问外网server用的（比如HTTP代理），从内到外。后来出现了反向代理，"反向"这个词在这儿的意思其实是指方向相反，即代理将来自外网client的请求forward到内网server，从外到内，反向代理服务器，对于客户端而言它就像是原始服务器。
&emsp;&emsp;两者的区别在于代理的对象不一样：正向代理代理的对象是客户端，反向代理代理的对象是服务端
&emsp;&emsp;我们常说的代理也就是指正向代理，正向代理的过程，它隐藏了真实的请求客户端，服务端不知道真实的客户端是谁，客户端请求的服务都被代理服务器代替来请求，某些科学上网工具扮演的就是典型的正向代理角色。用浏览器访问 http://www.google.com 时，被残忍的block，于是你可以在国外搭建一台代理服务器，让代理帮我去请求google.com，代理把请求返回的相应结构再返回给我。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-06.jpg" width="60%">

&emsp;&emsp;反向代理隐藏了真实的服务端，当我们请求 www.baidu.com 的时候，就像拨打10086一样，背后可能有成千上万台服务器为我们服务，但具体是哪一台，你不知道，也不需要知道，你只需要知道反向代理服务器是谁就好了，www.baidu.com 就是我们的反向代理服务器，反向代理服务器会帮我们把请求转发到真实的服务器那里去。Nginx就是性能非常好的反向代理服务器，用来做负载均衡。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-07.jpg" width="60%">

## 更好的理解反向代理
&emsp;&emsp;在计算机世界里，由于单个服务器的处理客户端（用户）请求能力有一个极限，当用户的接入请求蜂拥而入时，会造成服务器忙不过来的局面，可以使用多个服务器来共同分担成千上万的用户请求，这些服务器提供相同的服务，对于用户来说，根本感觉不到任何差别。
&emsp;&emsp;反向代理的实现：
1）需要有一个负载均衡设备来分发用户请求，将用户请求分发到空闲的服务器上（Nginx实际就是做这个的）
2）服务器返回自己的服务到负载均衡设备（比如从不同的tomcat返回结果）
3）负载均衡将服务器的服务返回用户（与客户的交互）

## 补充一个助攻图
&emsp;&emsp;正向代理中，proxy和client同属一个LAN，对server透明；反向代理中，proxy和server同属一个LAN，对client透明。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-08.jpg" width="60%">