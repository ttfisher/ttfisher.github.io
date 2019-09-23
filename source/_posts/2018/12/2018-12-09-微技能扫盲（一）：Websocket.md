---
title: 微技能扫盲（一）：Websocket（未完成...)
categories:
  - 多元化技能储备
  - 细微见真章
tags:
  - Websocket
comments: true
abbrlink: 4a027f14
date: 2018-12-10 09:48:43
---
【引言】Socket大多数人都知道，但是WebSocket是个什么鬼？简单粗暴的理解，它就是一组网络通信协议，跟我们平时用的http之类的类似，这个专题就来对WebSocket做一个初步的探索。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-12-09-01.jpg" width="55%"/></div>
<!-- more -->

# WebSocket 是什么？
&emsp;&emsp;通常WebSocket会被大家拿来和HTTP一起说，但是实际上它们虽然有些关系，却不是一个东西。
&emsp;&emsp;WebSocket实际上是HTML5出的东西（协议），是一种网络通信协议，Websocket其实是一个新协议，跟HTTP协议基本没有关系，只是为了兼容现有浏览器的握手规范看上去跟HTTP协议有些类似而已，也就是说它是HTTP协议上的一种补充。
&emsp;&emsp;借用下图（来自大神阮一峰的博客）来简单阐述一下，之所以WebSocket会诞生，就是因为HTTP是有缺陷的（单向，短连接）；所以WebSocket是为了解决HTTP原生是不支持持久连接的问题而诞生的（长连接，循环连接除外）；当然使用HTTP协议我们也可以通过轮询的方式保持伪长连接，但是如此一来资源消耗多不说，实现也很不够优雅。
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-12-26-02.jpg" width="50%">

# WebSocket 的特点
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-12-26-03.jpg" width="50%">
- 建立在 TCP 协议之上，服务器端的实现比较容易。
- 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
- 数据格式比较轻量，性能开销小，通信高效。
- 可以发送文本，也可以发送二进制数据。
- 没有同源限制，客户端可以与任意服务器通信。
- 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。（比如：ws://demo.com:80/myws）
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-12-26-04.jpg" width="40%">