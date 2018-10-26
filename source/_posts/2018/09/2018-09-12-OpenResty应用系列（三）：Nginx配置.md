---
title: OpenResty应用系列（三）：Nginx配置
categories:
  - 【302】如何从服务的前端优化负载
tags:
  - OpenResty
abbrlink: a67a2626
date: 2018-09-12 09:26:00
---
【引言】前一节讲到了Nginx的安装，实际使用时我们更关注的是配置，通过Nginx怎么实现反向代理？怎么实现负载均衡？里面还是有很多细节的，所以这一节就重点来说说配置的事情。
<div align=center><img src="/img/2018/2018-05-31-01.jpg" width="500"/></div>
<!-- more -->

# nginx.conf
【注】除了nginx.conf，其余配置文件，一般只需要使用默认提供即可。

## 基本结构
```
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;
        location / {
            default_type text/html;
            content_by_lua '
                ngx.say("<p>hello, world</p>")
            ';
        }
    }
}
```

- 全局块：配置允许运行的用户（组）、允许生成的工作进程数量、日志存放路径，pid存放路径以及其它配置文件引入等；对应上图就是events上面的部分；
- events块：配置是否同时接收多个链接以有及采用哪种驱动模型，每个工作进程支持的最大连接数等；
- http块：配置的主要部分，配置反向代理、缓存以及日志定义等绝大多数功能；
- server块: 这个是在http内部配置的，因为比较重要，所以单独拎出来；相当于一台虚拟主机，用户可以访问此虚拟主机的域名和端口；
- location块：这个是在server内部配置的，因为比较重要，所以也单独拎出来；相当于配置访问路由，对URI的上下文进行匹配映射；

## 全局块、events块
```
# user  nobody;
-- 定义Nginx运行的用户和用户组

worker_processes  1;
-- 表示工作进程的数量，一般设置为cpu的核数，也可以auto模式由程序自行控制

# error_log  logs/error.log;
# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;
-- 全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]

pid /var/run/nginx.pid;
-- 进程文件

events {
   worker_connections  1024;
   use epoll;
}
-- 工作模式（驱动模型）与连接数上限
-- worker_connections是单个后台worker process进程的最大并发链接数
-- 并发总数是 worker_processes 和 worker_connections 的乘积， 即 max_clients = worker_processes * worker_connections
```

## http块
```
    include mime.types;
--  文件扩展名与文件类型映射表

    default_type  application/octet-stream; 
-- 默认文件类型

log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
-- 日志格式定义

access_log  logs/access.log  main;
-- 日志路径

sendfile on;  
-- 开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，一般和tcp_nodelay on; 配合使用
-- 普通应用设为 on，如果用来进行下载等磁盘IO重负载应用，可设置 为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。
-- 注意:如果图片显示不正常 把这个改成off。

autoindex on; 
-- 开启目录列表访问，合适下载服务器，默认关闭。

tcp_nopush on; 
-- 防止网络阻塞

tcp_nodelay on; 
-- 防止网络阻塞

keepalive_timeout 120; 
-- 长连接超时时间，单位是秒

gzip on; 
-- 开启gzip压缩输出

server节点...
-- 虚拟主机节点，参考下一节
```

## server节点

### server节点的结构
```
# 可以在一个http块内配置多个server
server {
    listen       443 ssl;
    server_name  gcdd.koncendy.com;
    location /   {
        ......
```

### server怎么理解？
&emsp;&emsp;虚拟主机，就是把一台物理服务器划分成多个“虚拟”的服务器，每一个虚拟主机都可以有独立的域名和独立的目录。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-04.jpg" width="65%">

### server的3种配置方式

#### 基于ip的虚拟主机
```
#  (一个主机绑定多个ip地址) --应用：外部网站
server{
 listen       192.168.1.1:80;
 server_name  localhost;
}
server{
 listen       192.168.1.2:80;
 server_name  localhost;
}
```

#### 基于域名的虚拟主机
```
# 应用：公司内部网站，外部网站的管理后台
# 域名可以有多个，用空格隔开
server{
 listen       80;
 server_name  www.nginx1.com www.nginx2.com;
}
server{
 listen       80;
 server_name  www.nginx3.com;
}
```

#### 基于端口的虚拟主机
```
# listen不写ip的端口模式；几乎不用
server{
 listen       80;
 server_name  localhost;
}
server{
 listen       81;
 server_name  localhost;
}
```

## location节点

### 基本用法
&emsp;&emsp;location映射解析规则：location [ = | ~ | ~* | ^~ ] uri { ... }；具体可参考下图：
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-05.jpg" width="75%">

### 补充说明
- 首先匹配  = 
- 其次匹配  ^~ 
- 其次是按文件中顺序的正则匹配
- 最后是交给  /  通用匹配
- 当有匹配成功时候，停止匹配，按当前匹配规则处理请求
- 特殊情况：不需要继续匹配正则 location : 
- 当普通 location 前面指定了“ ^~ ”，特别告诉 Nginx 本条普 通 location 一旦匹配上，则不需要继续正则匹配。 
- 当普通location 恰好严格匹配上 ，不是最大前缀匹配，则不再继续匹配正则

## ReWrite

### 基本用法
&emsp;&emsp;rewrite    <regex>    <replacement>    [flag];（eg：rewrite ^/(.*)$ $raw_domain/$1 permanent;），从左至右各项的用途：
- 关键字：就是rewrite本身
- 正则表达式：根据此regex（正则表达式）匹配部分的内容
- 替代内容：替换正则表达式匹配到的内容
- flag标记：rewrite支持的flag标记

### 关于flag标记
- last – 基本上都用这个Flag
- break – 中止Rewirte，不在继续匹配
- redirect – 返回临时重定向的HTTP状态302
- permanent – 返回永久重定向的HTTP状态301

### 用来判断的表达式
- -f和!-f用来判断是否存在文件
- -d和!-d用来判断是否存在目录
- -e和!-e用来判断是否存在文件或目录
- -x和!-x用来判断文件是否可执行

### 补充说明
&emsp;&emsp;当在location区块中使用 if 指令的时候会有一些问题, 在某些情况下它并不按照你的预期运行而是做一些完全不同的事情. 而在另一些情况下他甚至会出现段错误. 一般来说避免使用 if 指令是个好主意.

# Nginx静态文件服务
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-10.jpg" width="60%">
&emsp;&emsp;我们只需要两个命令就可以启用基础缓存： proxy_cache_path 和 proxy_cache 。proxy_cache_path用来设置缓存的路径和配置，proxy_cache用来启用缓存（location配置： proxy_cache my_cache;）。如下示例：
```
proxy_cache_path/path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
```

- 缓存磁盘目录
- 设置了一个两级层次结构的目录
- 设置一个大小10M的共享内存区
- 缓存的上限10G
- 项目在不被访问的情况下能够在内存中保持的时间（60min）
- Nginx 最初会将注定写入缓存的文件先放入一个临时存储区域；use_temp_path=off命令指示 Nginx 将在缓存这些文件时将它们写入同一个目录下

# Nginx日志

## access_log
&emsp;&emsp;记录客户端访问Nginx的每一个请求，格式可以自定义；，语法: log_format name string; 其中name表示格式名称，string表示定义的格式字符串。log_format配置必须放在http内，否则会出现警告
access_log指令用来指定访问日志文件的存放路径（包含日志文件名）、格式和缓存大小，语法：access_log path [format_name [buffer=size | off]]; 其中path表示访问日志存放路径，format_name表示访问日志格式名称，buffer表示缓存大小，off表示关闭访问日志。
定义日志使用的字段及其作用：
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-11.jpg" width="75%">

## error_log
&emsp;&emsp;error_log主要记录客户端访问Nginx出错时的日志，格式不支持自定义。
&emsp;&emsp;error_log指令用来指定错误日志，语法: error_log path(存放路径) level(日志等级); 其中path表示错误日志存放路径，level表示错误日志等级，日志等级包括debug、info、notice、warn、error、crit，从左至右，日志详细程度逐级递减，即debug最详细，crit最少，默认为crit。

# 如何通过Nginx实现负载均衡？

## upstream
- upstream 是 Nginx 的 HTTP Upstream 模块，这个模块通过一个简单的调度算法来实现客户端 IP 到后端服务器的负载均衡。
- upstream 的名称可以任意指定，在后面需要用到的地方直接调用即可。
- upstream 支持的负载均衡算法：
 - 轮询（默认）。每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器宕机，故障系统被自动剔除，使用户访问不受影响。Weight 指定轮询权值，Weight值越大，分配到的访问机率越高，主要用于后端每个服务器性能不均的情况下。
 - ip_hash。每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后端服务器，有效解决了动态网页存在的session共享问题。
   -fair。这是比上面两个更加智能的负载均衡算法。此种算法可以依据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持fair的，如果需要使用这种调度算法，必须下载Nginx的upstream_fair模块。
 - url_hash。此方法按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率。Nginx本身是不支持url_hash的，如果需要使用这种调度算法，必须安装Nginx 的hash软件包。
 - least_conn。最少连接负载均衡算法，简单来说就是每次选择的后端都是当前最少连接的一个server(这个最少连接不是共享的，是每个worker都有自己的一个数组进行记录后端server的连接数)。*hash。这个hash模块又支持两种模式hash, 一种是普通的hash, 另一种是一致性hash(consistent)。
- upstream 支持的状态参数：
 - down，表示当前的server暂时不参与负载均衡。
 - backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是backup。
 - max_fails，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。
 - fail_timeout，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。

## 配置示例
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-12.jpg" width="65%">

# Nginx常用命令汇总
```
# 启动
nginx # 使用默认配置启动
nginx -c /etc/nginx/nginx.conf # 指定配置文件启动

# 停止
nginx -s stop
nginx -s quit
pkill -9 nginx

# 其他
nginx -h # 帮助 
nginx -v # 显示版本  
nginx -V # 显示版本和配置信息  
nginx -t # 测试配置  
nginx -q # 测试配置时，只输出错误信息  
nginx -s reload # 重新加载配置
```

# Nginx内置绑定变量列表
&emsp;&emsp;这里列举了Nginx内置绑定的变量，这些变量在Nginx实际运行时都会用来记录一些运行时的信息（比如一些请求地址、参数、客户端信息等），通常情况下，用到这些信息的机会不是太多，但在需要对请求进行一些处理（比如拦截、转发、复制等）时，还是需要用到这些变量的；比如：
- 日志打印时可以打印一些相关的信息，便于我们进行请求统计和问题排查
- 使用拦截策略配置时可以使用，比如在nginx端进行一些反爬策略的运作
- 使用Lua做一些处理时可以使用，比如对请求进行复制进行真实请求的测试引流
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-12-13.jpg" width="60%">

# nginx.conf示例
&emsp;&emsp;以下是在测试反爬策略时配置的简单的nginx调度转发，并未涉及很多配置项，仅供参考。
```
##########################################################################
# Description ： Nginx配置
# Author ： chenglin
# Date ： 2018-09-14
# Ana ： 基本策略：在所有请求上添加access_by_lua_file来进行访问控制
##########################################################################

# 这里用root的目的是为了执行lua脚本需要一定的权限
user root;
worker_processes  4;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    # lua库的地址（比如：redis）
    lua_package_path "/root/anti_spider/lib/?.lua;;";
    
    # 反爬的server
    server {
    
        # 监控域名和端口号
        listen       80;
        server_name  localhost;

        access_log  logs/host.access.80.log  main;
        
        # nginx自带的反爬策略文件位置
        include /usr/local/nginx/ext/anti_spider.conf;

        location / {
            default_type 'text/html';
            lua_code_cache off;
            access_by_lua_file /root/anti_spider/anti_spider.lua;
            proxy_pass http://localhost:8011/;
        }
    }

    # 跳转的server
    server {
    
        # 监控域名和端口号
        listen       8011;
        server_name  localhost;

        access_log  logs/host.access.8011.log  main;

        location / {
            # access_by_lua_file /root/anti_spider/anti_spider_internal.lua;
            
            # proxy_pass   http://gcddV2/gcdd-api/
            proxy_pass http://localhost:8080/;
        }
    }
}
```