---
title: OpenResty应用系列（五）：OpenRestry With Redis
categories:
  - Unprofessional Skills - Technical
tags:
  - OpenResty
abbrlink: 95a8ee39
date: 2018-09-12 19:26:00
---
【引言】既然是在OpenResty里面使用，Lua基本上是更多的参与到一些请求管理方面的工作，比如分发，比如请求复制，比如还能做一些反爬，说到反爬，实际可以结合Redis这个帮手来做，所以这一章我们先不纠结Lua本身的语法，先来看一个Redis操作的Demo，从实践中去理解Lua的基本语法。
<div align=center><img src="/img/2018/2018-09-13-05.jpg" width="500"/></div>
<!-- more -->

# Redis库引入
&emsp;&emsp;实际上，Lua和很多语言也有类似之处，比如它在较高版本后也支持模块和包，而对于Redis的操作，在Python里面我们有第三方的库可以引用，在SpringBoot里面也有相对应的Template，到了Lua里，实际也是有这么一个库的，这里是参考地址： https://github.com/openresty/lua-resty-redis 
&emsp;&emsp;这个Redis库下载到本地，解压完会有个lua-resty-redis-master目录，实际对我们比较有用的就是其中的lib，其他的暂时可以不关注。（<a target="new" href="/attaches/2018-09-13-redis.lua">点击下载redis.lua脚本</a>）；若通过此处下载的话，下载到本地后请将文件名修改为redis.lua。
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