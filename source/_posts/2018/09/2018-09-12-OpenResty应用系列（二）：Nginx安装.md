---
title: OpenResty应用系列（二）：Nginx安装
categories:
  - 分布式服务辅助之OpenResty
tags:
  - OpenResty
abbrlink: b2489ab
date: 2018-09-12 07:26:00
---
【引言】作为OpenResty的基石，Nginx是必须先了解的，所以本节就来说一说它的安装方法，其实和现在很多开箱即用的软件一样，Nginx也是基本上解压完配置配置就能用起来了。
<div align=center><img src="/img/2018/2018-09-13-03.jpg" width="500"/></div>
<!-- more -->

# windows下安装
&emsp;&emsp;到官网找到windows版本的安装包，然后下载到本地（官方下载地址： http://nginx.org/en/download.html ）；解压安装包，解压完结果如下：
```
D:\work_install\nginx-1.10.2>dir
驱动器 D 中的卷没有标签。
卷的序列号是 000B-9547

D:\work_install\nginx-1.10.2 的目录

2018/09/06  15:26    <DIR>          .
2018/09/06  15:26    <DIR>          ..
2018/09/06  15:10    <DIR>          conf
2018/04/27  16:10    <DIR>          contrib
2018/04/27  16:10    <DIR>          docs
2018/04/27  16:10    <DIR>          html
2018/09/07  09:17    <DIR>          logs
2016/10/18  17:45         2,905,088 nginx.exe
2018/04/27  16:13    <DIR>          temp
              1 个文件      2,905,088 字节
              8 个目录 43,858,440,192 可用字节
```

&emsp;&emsp;双击nginx.exe就可以运行nginx了，默认情况下不做任何配置直接启动也可以，这时会在本地80端口启动服务。
&emsp;&emsp;接下来本地浏览器访问：http://localhost/ ；出现如下展示信息的页面即表示启动成功（实际上是因为nginx默认启动在80端口，所以localhost后面的端口号是可以省略的）：
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-03.jpg" width="70%">

# Linux环境的安装
- 获取 Nginx源码包，在http://nginx.org/en/download.html上可以获取当前最新的版本
- 解压缩nginx-xx.tar.gz包
- 进入解压缩目录，执行./configure
- make & make install （需要预先安装gcc）
- 若安装时报找不到依赖模块，可使用--with-openssl= <openssl_dir> 、--with-pcre= <pcre_dir> 、--with-zlib= <zlib_dir> 指定依赖的模块目录。
- 验证方式同上一节：http://localhost/ 
【注】：可参考《运维技能集锦系列（一）：Nginx Keepalived环境搭建》

# CentOS安装实战记录
```
# 下载相关组件
[root@localhost src]# wget http://nginx.org/download/nginx-1.10.2.tar.gz
[root@localhost src]# wget http://www.openssl.org/source/openssl-fips-2.0.10.tar.gz
[root@localhost src]# wget http://zlib.net/zlib-1.2.11.tar.gz
[root@localhost src]# wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.40.tar.gz

# 安装c++编译环境，如已安装可略过
[root@localhost src]# yum install gcc-c++

# 期间会有确认提示输入y回车
Is this ok [y/N]:y

# openssl安装
[root@localhost src]# tar zxvf openssl-fips-2.0.10.tar.gz
[root@localhost src]# cd openssl-fips-2.0.10
[root@localhost openssl-fips-2.0.10]# ./config && make && make install

# pcre安装
[root@localhost src]# tar zxvf pcre-8.40.tar.gz
[root@localhost src]# cd pcre-8.40
[root@localhost pcre-8.40]# ./configure && make && make install

# zlib安装
[root@localhost src]# tar zxvf zlib-1.2.11.tar.gz
[root@localhost src]# cd zlib-1.2.11
[root@localhost zlib-1.2.11]# ./configure && make && make install

# nginx安装
[root@localhost src]# tar zxvf nginx-1.10.2.tar.gz
省略安装内容...
[root@localhost src]# cd nginx-1.10.2
[root@localhost nginx-1.10.2]# ./configure && make && make install

# 进入nginx目录并启动
[root@localhost src]# cd /usr/local/nginx/sbin/nginx
[root@localhost nginx]# nginx

# 访问http://localhost/ 进行验证，同上
```