---
title: OpenResty应用系列（四）：OpenResty环境搭建
categories:
  - 分布式服务辅助之OpenResty
tags:
  - OpenResty
abbrlink: 1cf451ee
date: 2018-09-12 17:26:00
---
【引言】前面几章主要是在讨论nginx，这一章开始终于要实实在在的接触到OpenResty了，跟很多常规的工具一样，我们必然首先要从安装说起。
<div align=center><img src="http://pm4hdun71.bkt.clouddn.com/img/public/000004.jpg" width="500"/></div>
<!-- more -->

# windows安装
&emsp;&emsp;下载地址： http://openresty.org/cn/download.html ；下载完成后解压到：D:\ProgramsGreen\openresty-1.13.6.1-win32；双击执行nginx.exe 或者使用命令start nginx启动nginx，如果没有错误现在nginx已经开始运行了；本地浏览器访问： http://localhost/ ；出现“Welcome to nginx!”页面即表示启动成功。

# CentOS yum安装
```
[root@localhost openresty]# yum install yum-utils
......
Complete!
[root@localhost openresty]# yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
已加载插件：fastestmirror, refresh-packagekit
adding repo from: https://openresty.org/package/centos/openresty.repo
grabbing file https://openresty.org/package/centos/openresty.repo to /etc/yum.repos.d/openresty.repo
openresty.repo                                                        |  267 B     00:00     
repo saved to /etc/yum.repos.d/openresty.repo
[root@localhost openresty]# yum install openresty
......
完毕！

# 至此已完成安装；为了便于启动，可以考虑修改一下环境变量（为了永久生效，最好还是直接修改/etc/profile）
[root@localhost openresty]# export PATH=$PATH:/opt/openresty/nginx/sbin追加到/etc/profile
[root@localhost openresty]# source /etc/profile
```

# CentOS编译安装
```
# 这个安装方式因为没有实际验证过，不保证完全准确
# 为了保险起见，先切换到root用户（后面的步骤也一样）

# 输入以下命令一次性安装需要的库；安装成功后会有“Complete！”字样。
[root@localhost conf]# yum install readline-devel pcre-devel openssl-devel perl 

# 下载OpenResty源码包
[root@localhost conf]# wget http://xxx/ngx_openresty-1.7.10.1.tar.gz

# 解压
[root@localhost conf]# tar xzvf ngx_openresty-1.7.10.1.tar.gz

# 配置
[root@localhost conf]# cd ngx_openresty-1.7.10.1
[root@localhost conf]# ./configure --prefix=/opt/openresty\  --with-luajit\ --without-http_redis2_module \  --with-http_iconv_module

# 编译
[root@localhost conf]# make
[root@localhost conf]# make install 
```

# 安装结果验证
&emsp;&emsp;在启动前，修改一下nginx.conf文件，加入了一句Lua脚本，内容如下（实际就是Lua版的Hello World）：
```
worker_processes  1;        #nginx worker 数量
error_log logs/error.log;   #指定错误日志文件路径
events {
    worker_connections 1024;
}
http {
    server {
        #监听端口，若你的6699端口已经被占用，则需要修改
        listen 6699;
        location / {
            default_type text/html;
            content_by_lua_block {
                ngx.say("HelloWorld")
            }
        }
    }
}
```
&emsp;&emsp;然后就开始启动nginx服务了（因为本机安装了普通版本的nginx，所以前面会有一个停服务的过程）。
```
[root@localhost conf]# pwd
/usr/local/openresty/nginx/conf
[root@localhost conf]# ps -ef|grep nginx
root      2352     1  0 14:07 ?        00:00:00 nginx: master process nginx
nobody    2396  2352  0 14:11 ?        00:00:00 nginx: worker process
root      3161  2398  0 17:20 pts/3    00:00:00 grep nginx
[root@localhost conf]# nginx -s stop
[root@localhost conf]# ps -ef|grep nginx
root      3164  2398  0 17:20 pts/3    00:00:00 grep nginx
[root@localhost conf]# /usr/local/openresty/nginx/sbin/nginx -p /usr/local/openresty/nginx/ -c conf/nginx.conf
[root@localhost conf]# ps -ef|grep nginx
root      3176     1  0 17:23 ?        00:00:00 nginx: master process /usr/local/openresty/nginx/sbin/nginx -p /usr/local/openresty/nginx/ -c conf/nginx.conf
nobody    3177  3176  0 17:23 ?        00:00:00 nginx: worker process                                                                  
root      3179  2398  0 17:23 pts/3    00:00:00 grep nginx
[root@localhost conf]# 
```
&emsp;&emsp;通过 http://localhost/ 访问，页面返回信息如下即表示安装验证成功；顺便补充说明一下，nginx启动时-p指定了启动的工作目录，-c指定了启动使用的配置文件，其实实际使用自定义配置和脚本时是可以单独拉一个目录出来做的。
<img style="clear: both;display: block;margin:auto;" src="http://pm4hdun71.bkt.clouddn.com/img/2018/2018-09-12-14.jpg" width="60%">

# LuaJIT
&emsp;&emsp;因为OpenResty的一大特点就是用到了Lua，而我们做Lua的开发肯定必不可少的需要做一些调试操作，所以在本机安装一个Lua环境也是必要的。
&emsp;&emsp;那么为什么这节的标题是LuaJIT呢？首先lua其实就是为了嵌入其它应用程序而开发的一个脚本语言，而LuaJIT是lua的一个Just-In-Time也就是运行时编译器，也可以说是lua的一个高效版，所以我们选择了它。Lua和LuaJIT的对比如下：
- Lua非常高效，它比许多其它脚本(如Perl、Python、Ruby)都快，尽管如此，仍然有人不满足。
- LuaJIT就是一个为了再榨出一点速度的尝试，它利用JIT编译技术把Lua代码编译成本地机器码后交由CPU直接执行。
- LuaJIT 是采用 C 语言写的 Lua 的解释器。LuaJIT 被设计成全兼容标准 Lua 5.1, 因此 LuaJIT 代码的语法和标准 Lua 的语法没多大区别。
- LuaJIT 和 Lua 的区别就是LuaJIT 的运行速度比标准 Lua 快数十倍；可以当做Lua 的高效率版本。
- LuaJIT 编译安装比较复杂，所以这里附上一个编译好的版本，亲测可用（ <a target="new" href="http://pm4hdun71.bkt.clouddn.com/attaches/2018-09-13-LuaJIT.zip">点击下载编译好的windows版本LuaJIT.zip</a> ）
