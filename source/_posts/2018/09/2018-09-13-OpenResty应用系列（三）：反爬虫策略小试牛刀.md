---
title: OpenResty应用系列（三）：反爬虫策略小试牛刀
categories:
  - 分布式服务
  - OpenResty
tags:
  - OpenResty
abbrlink: 5e12f98d
date: 2018-09-13 09:26:00
---
【引言】其实前一节引入Lua操作Redis的应用意图是很明显的，也是为了本节做一个铺垫，有了Redis，我们可以很方便的记录很多信息，而且Redis的自动过期特性对我们做反爬监控也是如虎添翼，所以本章我们基于前面的积累来探讨一下反爬的策略，当然还很粗浅，只是实现了一些非常基础的反爬策略。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/public/000025.jpg" width="55%"/></div>
<!-- more -->

# 基本思路
&emsp;&emsp;首先，nginx自带了反爬的一些基础配置，虽然功能不是那么强大，但是起码可以防止一些低级爬虫的出现，所以在这里首先在nginx侧做了一份基本的配置（anti_spider.conf）。
&emsp;&emsp;其次，因为需要拦截爬虫，那就需要对所有的请求进行统计，所以在nginx配置时对外端口对所有请求进行拦截，走lua进行访问频率统计，若出现疑似爬虫，则在这一层进行拦截（anti_spider.lua）。在通过第一层拦截后，会进入第二个server，这个server的控制比较简单，也就是限制本端口只能通过localhost进行访问，避免外部主机通过IP+Port方式绕过第一层。
&emsp;&emsp;本次反爬试验的实现的思路其实还比较简单，拦截的方式也比较粗暴（按1分钟的访问数量进行拦截的），后面实际应用是肯定还需要进行一些优化的（比如：统计不同时间段的访问频度，统计访问的API类型是否有共性，等等），路漫漫其修远兮，这里小试牛刀，所有脚本也正常通过验证，再次记下一笔，后续如有优化升级再做记录补充。
&emsp;&emsp;鉴于本人在这块儿也是新手，属于边学边用的状态，如有什么使用或描述不当的地方，后续发现再做更正。

# 脚本概览
&emsp;&emsp;本次试验主要涉及以下4个脚本：
- nginx.conf：无需多言了，nginx的默认配置文件嘛
- anti_spider.conf：这个文件定义的就是nginx自带的一些基础反爬检测策略
- anti_spider.lua：这个就是第一层全量请求统计检测的脚本
- anti_spider_internal.lua：这个是针对内部跳转后的server的localhost控制

```
C:\Users\Administrator\Desktop\反爬策略\实用版>tree /f
文件夹 PATH 列表
卷序列号为 34E0-D97F
C:.
│  anti_spider.conf
│  anti_spider.lua
│  anti_spider_internal.lua
│  nginx.conf
│
└─lib
    └─resty
            redis.lua

C:\Users\Administrator\Desktop\反爬策略\实用版>
```

# anti_spider.conf
```
##########################################################################
# Description ： Nginx自带的反爬功能，虽然比较弱，总比没有好
# Author ： chenglin
# Date ： 2018-09-14
# Ana ： - 第一层策略使用user-agent，拦截比较低级的爬虫
#        - 第二层根据请求的方式，屏蔽异常方式的请求
#        - 第三层可根据Nginx的日志分析人工补充异常IP，亡羊补牢之策
##########################################################################

# 禁止一般的爬虫工具（实际上稍微有点水平的爬虫都不会傻到用默认UA的）
if ($http_user_agent ~* (Scrapy|Curl|HttpClient)) {  
     return 403;  
}

# 禁止指定UA及UA为空的访问  
if ($http_user_agent ~ "WinHttp|WebZIP|FetchURL|node-superagent|java/|FeedDemon|Jullo|JikeSpider|Indy Library|Alexa Toolbar|AskTbFXTV|AhrefsBot|CrawlDaddy|Java|Feedly|Apache-HttpAsyncClient|UniversalFeedParser|ApacheBench|Microsoft URL Control|Swiftbot|ZmEu|oBot|jaunty|Python-urllib|lightDeckReports Bot|YYSpider|DigExt|HttpClient|MJ12bot|heritrix|EasouSpider|Ezooms|BOT/0.1|YandexBot|FlightDeckReports|Linguee Bot|^$" ) {  
     return 403;
}

# 禁止非GET|POST方式的抓取  
if ($request_method !~ ^(GET|POST)$) {  
    return 403;  
}

# 禁止某IP/IP段的访问（因为现在还没有发现异常IP，后期若有发现可以手工添加）
# deny 58.95.66.0/24;
```

# nginx.conf
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
            
            # proxy_pass   http://demoV2/demo-api/
            proxy_pass http://localhost:8080/;
        }
    }
}
```

# anti_spider.lua
```lua
------------------------------------------------------------------------------------
-- Description ： 根据客户端的访问频率进行拦截，出发点是为了避免被爬虫入侵
-- Author ： chenglin
-- Date ： 2018-09-13
-- Ana ： - 当某个周期内超过访问频率，则标记封禁一个单位
--        - 一次解封后，记录上次封禁时长，下次若再触发封禁，封禁时间增加一个单位
--        - 封禁时长的有效期是可配置的（默认是一天）
------------------------------------------------------------------------------------

------------------------------------------------------------------------------------
-- Part-0：常量部分
------------------------------------------------------------------------------------

-- 日志文件路径；日志记录开关
local log_path = '/root/anti_spider/anti_spider.log'
local log_switch = 1

-- Redis连接IP和端口号
local redis_ip = '127.0.0.1'
local redis_port = '6379'

-- 封禁指标：一段时间内内达到一定的访问次数就禁止（比如：每60秒10次以上）
local monitor_cycle = 60 
local monitor_toplimit = 10 

-- 单次禁入时长（多次触发封禁会线性递增）
local ban_time_unit = 600 

-- 禁入时间缓存周期（禁入周期以天为单位，隔天归零）
local ban_time_expire = 86400 

-- 被封的时间周期在redis的key
local key_ban_time_by_ip = 'ban_time_' .. ngx.var.remote_addr

-- 被封的标志在redis的key
local key_ban_flag_by_ip = 'ban_flag_' .. ngx.var.remote_addr

-- 所有访问过的IP在redis的key
local key_access_total_ip = 'access_total_' .. os.date('%x')

-- 计时起始点在redis的key
local key_timing_start_by_ip = 'monitor_start_' .. ngx.var.remote_addr

-- 计时周期内的访问次数在redis的key
local key_access_count_by_ip = 'access_count_' .. ngx.var.remote_addr

-- 所有被封过的IP在redis的key
local key_banned_total = 'banned_total_' .. os.date('%x')

------------------------------------------------------------------------------------
-- Part-1.1：redis连接操作方法部分
------------------------------------------------------------------------------------

-- 连接redis
local function conn_redis()
    -- 连接redis，同时设置连接超时时间
    local m_redis = require 'resty.redis' 
    local redis = m_redis.new() 
    local ok ,err = redis.connect(redis, redis_ip, redis_port) 
    redis:set_timeout(60000) 

    -- 连接Redis失败
    if not ok then 
        ngx.say("Error : ",err)
        close_redis(redis) 
        return nil
    end
    
    return redis
end

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

------------------------------------------------------------------------------------
-- Part-1.2：记录日志的公共方法
------------------------------------------------------------------------------------

-- 写几句日志，调试时用的
local function write_log(content)
    
    -- 开关判断
    if tonumber(log_switch) == 1 then 
    
        -- 追加方式打开文件
        local file = io.open(log_path, "a")
        if file then
            
            -- 写文件
            if file:write(content .. "\n") == nil then 
                return false 
            end
            
            -- 关闭流
            io.close(file)
            return true
        end
    end
    return false
end


------------------------------------------------------------------------------------
-- Part-2：获取redis连接和一些初始化操作
------------------------------------------------------------------------------------

local redis_conn = conn_redis()
if not redis_conn then 
    ngx.say("Error : ",err)
    return nil
end

-- 统计当日访问ip集合（通过set的不重复性实现）；可以通过SMEMBERS statistic_total_ip:09/13/18查看
redis_conn:sadd(key_access_total_ip, ngx.var.remote_addr) 


------------------------------------------------------------------------------------
-- Part-3.1：黑白名单校验
------------------------------------------------------------------------------------

-- Redis白名单（set）
local is_white ,err = redis_conn:sismember('white_list', ngx.var.remote_addr) 
if is_white == 1 then 
    return 
end 

-- Redis黑名单（set） 
local is_black ,err = redis_conn:sismember('black_list', ngx.var.remote_addr) 
if is_black == 1 then 
    close_redis(redis);
    ngx.exit(ngx.HTTP_FORBIDDEN) 
end 

------------------------------------------------------------------------------------
-- Part-3.2：确定是否已经被封禁，如果已经被封禁，则走封禁逻辑并结束流程
------------------------------------------------------------------------------------

-- 查询ip是否已经被封禁，若已经被封禁，则做封禁处理（验证码 ，403等）
local is_banned , err = redis_conn:get(key_ban_flag_by_ip) 
    
-- 被封了
if is_banned and tonumber(is_banned) == 1 then 
    
    -- 原始请求
    -- local origin_request = ngx.encode_base64(ngx.var.scheme .. '://' .. ngx.var.host .. ':' .. ngx.var.server_port .. ngx.var.request_uri) 
    -- 推荐插件：极验，可以通过如下dest请求进行验证
    -- local dest = 'http://127.0.0.1:5000/' .. '?continue=' .. origin_request 
    -- ngx.redirect(dest,302) 
    
    -- 别忘了释放redis连接
    close_redis(redis);
    
    -- 跳转到封禁页面，结束；这里直接返回403
    ngx.exit(ngx.HTTP_FORBIDDEN)
end 

------------------------------------------------------------------------------------
-- Part-4：获取redis已有的封禁数据，没有就初始化为一个封禁周期
------------------------------------------------------------------------------------

-- redis中本IP下一轮封禁的时长（秒）
local ban_time, err = redis_conn:get(key_ban_time_by_ip) 

if ban_time == ngx.null then 
    
    -- 没设置的话就设置一个封禁单位
    redis_conn:set(key_ban_time_by_ip, ban_time_unit) 
    
    -- 设置redis中的封禁时间expire（在没有达到expire有效期内，每封禁一次，时长增加一个单位）
    redis_conn:expire(key_ban_time_by_ip, ban_time_expire) 
end 

------------------------------------------------------------------------------------
-- Part-5：如果还未被封禁，则需要累计当前请求数，并判断是否达到封禁要求
------------------------------------------------------------------------------------

-- ip监控起始时间
local monitor_start , err = redis_conn:get(key_timing_start_by_ip) 

-- 还没记录过（新请求）、超过一个监控周期（monitor_cycle）；则启动一轮新的监控
if monitor_start == ngx.null or os.time() - tonumber(monitor_start) > monitor_cycle then 

    -- 起始时间为当前时间
    redis_conn:set(key_timing_start_by_ip , os.time()) 

    -- 访问量为1
    redis_conn:set(key_access_count_by_ip , 1) 
else 

    -- 访问量（基于IP的）
    local access_count , err = redis_conn:get(key_access_count_by_ip) 

    -- 在局部变量记录本次访问
    access_count = access_count + 1 
    
    write_log("Access once ....")
    
    -- 向redis记录本次访问
    redis_conn:incr(key_access_count_by_ip) 
    
    -- 达到封禁条件
    if access_count >= monitor_toplimit then 
    
        -- 封禁一个IP，记录封禁标志
        redis_conn:set(key_ban_flag_by_ip , 1) 
        
        -- 设置封禁过期时间（到期redis直接expire就解禁了）
        redis_conn:expire(key_ban_flag_by_ip , ban_time) 
        
        -- 每被封禁一次，封禁时间延长一个周期（也可以更狠一些，每轮翻倍之类的）
        redis_conn:incrby(key_ban_time_by_ip, ban_time_unit) 
        
        -- 统计当日屏蔽ip总数 
        redis_conn:sadd(key_banned_total, ngx.var.remote_addr) 
    end 
end 

------------------------------------------------------------------------------------
-- Part-6：没有被封禁就继续，这个由nginx的配置来控制，所以下面的代码没用了
------------------------------------------------------------------------------------

-- 获取请求参数；再次发起请求
-- local action = ngx.var.request_method
-- if action == "POST" then
--     ngx.location.capture(
--         "/smart/" .. ngx.var.request_uri,
--         { method = ngx.HTTP_POST, body = ngx.req.read_body() }
--     )
-- else
--     ngx.location.capture(
--         "/smart/" .. ngx.var.request_uri,
--         { method = ngx.HTTP_GET}
--     )
-- end

-- ngx.say("This is normal page ......")
-- ngx.redirect(ngx.var.scheme .. '://' .. ngx.var.host .. ':' .. ngx.var.server_port .. uri, 301)
-- ngx.req.set_uri(ngx.var.scheme .. '://' .. ngx.var.host .. ':' .. ngx.var.server_port .. uri, true);  

------------------------------------------------------------------------------------
-- The End
------------------------------------------------------------------------------------
```

# anti_spider_internal.lua
```lua
if ngx.var.host ~= "localhost"  then  
    ngx.exit(ngx.HTTP_FORBIDDEN)
end  
```

# 成果验证
&emsp;&emsp;本次针对反爬策略的验证，使用的是tomcat自带的example来进行的，通过浏览器多次访问后触发拦截限制，会出现403的现象，实际证明拦截策略生效。
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-09-13-07.jpg" width="60%">

# 结语
&emsp;&emsp;此次试验仅简单的针对同一IP的高频度访问进行了拦截，效果是实现了，但是距离实际使用可能还有一段距离；因为实际业务场景中，爬虫远不会如此简单粗暴，所以后面还是要考虑从其他层面来进行拦截策略的补充；所以后面会有个大展拳脚的专题来对反爬进行升级优化，虽然理论上爬虫是无法完全被检测到的，但是在能力范围内能做的尽量还是要去做到。
&emsp;&emsp;下一步的反爬策略升级的方向（当然这里提到的策略还并不完善，还有待补充形成一套成熟的方案）：
- 比如基于URL的类型进行重点拦截，往往最担心被爬虫获取的是检索列表页和详情页，所以可以考虑对这两种接口的拦截策略进行定制
- 比如基于访问路线的分析，这个可能比较困难，也就是说针对不是通过常规路径进入系统（比如没有列表访问请求直接访问详情页）的情况，进行专门的定制拦截