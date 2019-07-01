---
title: Java Virtual Machine探秘（零一）：Java内存模型
categories:
  - 技术结构升级之JVM微专题
tags:
  - JVM
abbrlink: 1d3f0047
date: 2019-07-01 07:21:56
---
【引言】Java Virtual Machine对于做Java的同学来说，一直是个熟悉又神秘的领域，所以这次抱着从零开始的心态从基础开始来探探JVM里面的秘密；第一篇就来说说Java Memory Model。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2019/2019-06-24-03.jpg" width="55%"/></div>
<!-- more -->

# N