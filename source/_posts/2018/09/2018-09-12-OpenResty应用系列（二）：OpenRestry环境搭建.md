---
title: OpenResty应用系列（二）：OpenResty环境搭建
categories:
  - 分布式服务辅助之OpenResty
tags:
  - OpenResty
abbrlink: 1cf451ee
date: 2018-09-12 17:26:00
---
【引言】前面几章主要是在讨论nginx，这一章开始终于要实实在在的接触到OpenResty了，跟很多常规的工具一样，我们必然首先要从安装说起。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/public/000004.jpg" width="55%"/></div>
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
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-09-12-14.jpg" width="60%">

# LuaJIT
&emsp;&emsp;因为OpenResty的一大特点就是用到了Lua，而我们做Lua的开发肯定必不可少的需要做一些调试操作，所以在本机安装一个Lua环境也是必要的。
&emsp;&emsp;那么为什么这节的标题是LuaJIT呢？首先lua其实就是为了嵌入其它应用程序而开发的一个脚本语言，而LuaJIT是lua的一个Just-In-Time也就是运行时编译器，也可以说是lua的一个高效版，所以我们选择了它。Lua和LuaJIT的对比如下：
- Lua非常高效，它比许多其它脚本(如Perl、Python、Ruby)都快，尽管如此，仍然有人不满足。
- LuaJIT就是一个为了再榨出一点速度的尝试，它利用JIT编译技术把Lua代码编译成本地机器码后交由CPU直接执行。
- LuaJIT 是采用 C 语言写的 Lua 的解释器。LuaJIT 被设计成全兼容标准 Lua 5.1, 因此 LuaJIT 代码的语法和标准 Lua 的语法没多大区别。
- LuaJIT 和 Lua 的区别就是LuaJIT 的运行速度比标准 Lua 快数十倍；可以当做Lua 的高效率版本。
- LuaJIT 编译安装比较复杂，所以这里附上一个编译好的版本，亲测可用（ <a target="new" href="http://pm4hdun71.bkt.clouddn.com/attaches/2018-09-13-LuaJIT.zip">点击下载编译好的windows版本LuaJIT.zip</a> ）

# Redis库引入
&emsp;&emsp;实际上，Lua和很多语言也有类似之处，比如它在较高版本后也支持模块和包，而对于Redis的操作，在Python里面我们有第三方的库可以引用，在SpringBoot里面也有相对应的Template，到了Lua里，实际也是有这么一个库的，这里是参考地址： https://github.com/openresty/lua-resty-redis 
&emsp;&emsp;这个Redis库下载到本地，解压完会有个lua-resty-redis-master目录，实际对我们比较有用的就是其中的lib，其他的暂时可以不关注。（redis.lua的下载地址： https://github.com/openresty/lua-resty-redis/tree/master/lib/resty ）
&emsp;&emsp;随后我们将lib上传到已经安装好OpenResty环境的服务器的某个路径下（特别要注意的是，要对所有需要执行的脚本进行赋权操作，一般755权限就够用了）：
```
[root@localhost openresty]# chmod 755 -R *
[root@localhost openresty]# ls -R -lrt
.:
总用量 8
-rwxr-xr-x 1 root root 1529 9月  13 10:40 test_redis.lua
drwxr-xr-x 3 root root 4096 9月  13 10:40 lib

./lib:
总用量 4
drwxr-xr-x 2 root root 4096 9月  13 10:40 resty

./lib/resty:
总用量 12
-rwxr-xr-x 1 root root 9250 9月  13 10:40 redis.lua
[root@localhost openresty]# 

```

# nginx.conf配置全貌
&emsp;&emsp;因为OpenResty里面Lua是依赖nginx来触发运行的，所以首先我们要搞定一套OK的nginx配置，保证nginx服务可以正常提供访问。
```
[root@localhost conf]# pwd
/usr/local/openresty/nginx/conf
[root@localhost conf]#
[root@localhost conf]# cat nginx.conf

# 注意点1
#user  nobody;
user root;
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    lua_package_path "/root/openresty/lib/?.lua;;";
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;
        #access_log  logs/host.access.log  main;
    #include /usr/local/nginx/ext/anti_spider.conf;

        location / {
            default_type 'text/html';
            lua_code_cache on;
            content_by_lua_file /root/openresty/test_redis.lua;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
[root@localhost conf]# 
```

# nginx.conf配置详述
&emsp;&emsp;这里我将我测试时用到的配置文件原封不动的贴过来了，一来比较完整，二来里面对应的部分可以做充分的解释理解；实际上主要是3个地方需要注意的，下面将一一道来。

## user
&emsp;&emsp;为什么要配置这个user为root呢？因为默认配置文件是没有配置user的，这时候启动nginx的用户默认就是nobody，而nobody是没有执行lua脚本权限的，所以会一直报错
```
# 1. 因为浏览器访问nginx服务总是404，所以我们去后台查看了一下nginx的error.log，发现如下日志：
2018/09/13 10:54:44 [error] 9654#9654: *2 failed to load external Lua file "/root/openresty/test_redis.lua": 
cannot open /root/openresty/test_redis.lua: Permission denied, client: 192.168.1.9, 
server: localhost, request: "GET /lua_test HTTP/1.1", host: "192.168.1.168"

# 2. 然后想到去修改一下Lua脚本的权限；结果发现还是一样的结果
[root@localhost openresty]# chmod 755 -R *

# 3. 于是通过进程运行用户查看发现nginx是nobody用户启动的
[root@localhost ~]# ps -ef|grep nginx
root      9574     1  0 10:49 ?        00:00:00 nginx: master process nginx -p /usr/local/openresty/nginx/ -c conf/nginx.conf
nobody    9575  9574  0 10:49 ?        00:00:00 nginx: worker process                                  
root      9576  9540  0 10:49 pts/2    00:00:00 vi nginx.conf
root      9580  9500  0 10:50 pts/1    00:00:00 grep nginx
[root@localhost ~]#

# 4. 于是修改了一下nginx.conf将用户改为root，然后重启nginx服务
#user  nobody;
user root;

# 5. 重新查看发现用户确实是root
[root@localhost ~]# ps -ef|grep nginx
root      9668     1  0 10:57 ?        00:00:00 nginx: master process nginx -p /usr/local/openresty/nginx/ -c conf/nginx.conf
root      9669  9668  0 10:57 ?        00:00:00 nginx: worker process                                  
root      9671  9500  0 10:58 pts/1    00:00:00 grep nginx
[root@localhost ~]# 

# 6. 最后通过curl测试一下，访问正常
[root@localhost openresty]# curl "http://localhost/lua"
ngs.say : this is redis speaking
[root@localhost openresty]#
```

## lua_package_path
&emsp;&emsp;这个地方其实可说的不多，参照着配置就可以了，基本通过命名也很好理解，就是配置Lua引入的包路径（也就是刚刚我们上传依赖库的路径，其实和很多语言的package引入的原理都是类似的）。

## location
&emsp;&emsp;这个节点有两个配置：一个是lua_code_cache，用来控制lua脚本缓存的，若设置为off则nginx不缓存lua脚本（但是启动nginx时会有alert信息出现，如下），每次改变lua代码，不必reload nginx，比较便于调试；但off实际上对性能有影响，故正式环境下一定记得设置为on；另外一个是content_by_lua_file，实际上类似的配置有好几种方式（后面再一一说明），这个地方就是以文件的方式引入；所以按照上面的配置，只要没有匹配到合适规则的url，都会走到/root/openresty/test_redis.lua这个文件里面去完成后续处理。
```
[root@localhost ~]# nginx -p /usr/local/openresty/nginx/ -c conf/nginx.conf
nginx: [alert] lua_code_cache is off; this will hurt performance in /usr/local/openresty/nginx/conf/nginx.conf:40
[root@localhost ~]#
```

# Lua脚本
&emsp;&emsp;这里我将测试的脚本完整的附在这边，有一个粗略的概念，并不要深究每句语法的细节，具体的语法细节后面会逐渐在实践中积累总结。
```lua
-- 关闭redis连接
local function close_redis(redis)
    if not redis then
        return
    end
    local ok,err = redis:close();
    if not ok then
        ngx.say("Error : ",err);
    end
end

-- 引入redis的module
local m_redis = require("resty.redis");

-- 创建redis实例
local redis = m_redis:new();

-- 设置操作的超时（ms）时间，包括连接也会遵循这个时间限制
redis:set_timeout(1000)

-- 建立到本地redis的连接
local ok,err = redis:connect('127.0.0.1',6379)

-- 连接如果不Ok，那么就给前端返回一句error信息，然后关闭连接
if not ok then
    ngx.say("Error : ",err)
    return close_redis(redis);
end

-- Redis的key
local key = "k1"

-- 设置一个key和对应的消息到redis，如果返回不正常就给前端返回错误信息
local resp,err = redis:set(key, "this is redis speaking")
if not resp then
    ngx.say("Error : ",err)
    return close_redis(redis)
end

-- 获取redis存储的消息
local resp, err = redis:get(key) 

-- 没响应
if not resp then  
    ngx.say("Error : ", err)
    return close_redis(redis)
end 

-- 值为空
if resp == ngx.null then
    resp = 'No data found in redis'
end

-- 正常返回
ngx.say("ngs.say : ", resp)  

-- 关闭连接
close_redis(redis)
```