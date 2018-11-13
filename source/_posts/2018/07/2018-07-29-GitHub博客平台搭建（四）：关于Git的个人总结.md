---
title: GitHub博客平台搭建（四）：关于Git的个人总结（未完成...)
categories:
  - 多元化技能储备之Git&GitHub
tags:
  - GitHub
abbrlink: f743d1c
date: 2018-07-29 17:17:00
---
【引言】为什么要写这么一篇总结呢？一来是为了圆了前面那一篇的说辞；二来是因为零零散散的接触Git也有些时日了，谈起Git竟不知道从何谈起，说不出个一二三来，所以，抽点时间整理一下，也纯当给自己的学习做个总结吧！（部分资源参考自网络）
<div align=center><img src="/img/2018/2018-07-29-01.jpg" width="500"/></div>
<!-- more -->

# Git是什么？
&emsp;&emsp;Git是目前世界上最先进的分布式版本控制系统（没有之一）。
&emsp;&emsp;版本控制最主要的功能就是追踪文件的变更。它将什么时候、什么人更改了文件的什么内容等信息忠实地了已录下来。每一次文件的改变，文件的版本号都将增加。除了记录版本变更外，版本控制的另一个重要功能是并行开发。

# Git的诞生趣事
&emsp;&emsp;很多人都知道，Linus在1991年创建了开源的Linux，从此，Linux系统不断发展，已经成为最大的服务器系统软件了。Linus虽然创建了Linux，但Linux的壮大是靠全世界热心的志愿者参与的，这么多人在世界各地为Linux编写代码，那Linux的代码是如何管理的呢？事实是，在2002年以前，世界各地的志愿者把源代码文件通过diff的方式发给Linus，然后由Linus本人通过手工方式合并代码！
&emsp;&emsp;你也许会想，为什么Linus不把Linux代码放到版本控制系统里呢？不是有CVS、SVN这些免费的版本控制系统吗？因为Linus坚定地反对CVS和SVN，这些集中式的版本控制系统不但速度慢，而且必须联网才能使用。有一些商用的版本控制系统，虽然比CVS、SVN好用，但那是付费的，和Linux的开源精神不符。
&emsp;&emsp;不过，到了2002年，Linux系统已经发展了十年了，代码库之大让Linus很难继续通过手工方式管理了，社区的弟兄们也对这种方式表达了强烈不满，于是Linus选择了一个商业的版本控制系统BitKeeper，BitKeeper的东家BitMover公司出于人道主义精神，授权Linux社区免费使用这个版本控制系统。安定团结的大好局面在2005年就被打破了，原因是Linux社区牛人聚集，不免沾染了一些梁山好汉的江湖习气。开发Samba的Andrew试图破解BitKeeper的协议（这么干的其实也不只他一个），被BitMover公司发现了（监控工作做得不错！），于是BitMover公司怒了，要收回Linux社区的免费使用权。
&emsp;&emsp;Linus可以向BitMover公司道个歉，保证以后严格管教弟兄们，嗯，这是不可能的。实际情况是这样的：Linus花了两周时间自己用C写了一个分布式版本控制系统，这就是Git！一个月之内，Linux系统的源码已经由Git管理了！牛是怎么定义的呢？大家可以体会一下。Git迅速成为最流行的分布式版本控制系统，尤其是2008年，GitHub网站上线了，它为开源项目免费提供Git存储，无数开源项目开始迁移至GitHub，包括jQuery，PHP，Ruby等等。
&emsp;&emsp;历史就是这么偶然，如果不是当年BitMover公司威胁Linux社区，可能现在我们就没有免费而超级好用的Git了。
> 注：本段落内容摘自大牛廖雪峰的Git教程，感谢有许许多多这么有奉献精神的大牛。

# Git的安装
&emsp;&emsp;下载地址：https://git-scm.com/downloads ；下载完成后在本机安装（具体下载什么版本就看电脑操作系统和位数了）；一路Next，安装完成后就会有一个Git Bash的程序，正常运行即表示安装成功了；安装完成后，按照下面的命令配置个人的用户名和邮箱（对了，在这之前最好是先注册一个GitHub的账号，具体的可参考前一篇文章）
```
// 填写你的github用户名
$ git config --global user.name "your user name" 

// 填写你的github注册邮箱
$ git config --global user.email "your email address" 
```
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-07-27-08.jpg" width="75%">

