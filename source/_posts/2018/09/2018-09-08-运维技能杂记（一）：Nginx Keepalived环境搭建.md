---
title: 运维技能杂记（一）：Nginx Keepalived环境搭建
categories:
  - 架构师野蛮生长之系统运维
tags:
  - Nginx
  - Keepalived
abbrlink: ae7da3ed
date: 2018-09-08 10:03:00
---
【引言】借着此次用到了Nginx和Keepalived来实现负载均衡的双机热备的机会，整理一下基本的安装和配置，很多东西都是在使用中熟知的，但是没多久就会遗忘，所以还是写几笔吧！
<div align=center><img src="/img/2018/2018-09-13-01.jpg" width="500"/></div>
<!-- more -->

# 概念

## Nginx是什么？
&emsp;&emsp;nginx [engine x] is an HTTP and reverse proxy server, a mail proxy server, and a generic TCP/UDP proxy server, originally written by Igor Sysoev. 通常情况下，我们是把nginx作为反向代理来使用的，当然它还有很多其他的作用，比如做web服务器，邮件服务器等等。

## keepalived是什么？
&emsp;&emsp;Keepalived is a routing software written in C. The main goal of this project is to provide simple and robust facilities for loadbalancing and high-availability to Linux system and Linux based infrastructures. 
&emsp;&emsp;Keepalived的作用是检测服务器的状态，如果有一台web服务器宕机，或工作出现故障，Keepalived将检测到，并将有故障的服务器从系统中剔除，同时使用其他服务器代替该服务器的工作，当服务器工作正常后Keepalived自动将服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的服务器。当然，针对服务的心跳检测来说也是一个道理。

# Nginx安装

## 下载安装源
> wget http://nginx.org/download/nginx-1.14.0.tar.gz

## 编译环境准备
```
# GCC编译环境
yum install gcc-c++

#  nginx的http模块使用pcre来解析正则表达式
yum install -y pcre pcre-devel

# 压缩和解压缩库
yum install -y zlib zlib-devel

# https依赖于openssl
yum install -y openssl openssl-devel
```

## 配置和编译
```
# 解压
tar -zxvf nginx-1.14.0.tar.gz

# 配置（需要事先建立好/var/temp/nginx/路径）
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi

# 编译
make
make install
```

## 启动服务
```
[root@localhost ~]# cd /usr/local/nginx/sbin/
[root@localhost sbin]# ./nginx 
[root@localhost sbin]# 
```

## 验证服务
&emsp;&emsp;经过上面的一系列步骤之后，Nginx应该都已经正常安装完成了，默认配置中Nginx是绑定80端口的，所以直接通过IP访问即可验证Nginx是否正常提供服务。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-08-01.jpg" width="70%">

# Keepalived安装

## yum安装
```
# 可以通过此命令直接安装，也可以通过后面复杂一些的方式安装
yum -y install keepalived
```

## 编译安装
```
# 因为yum安装相对简单，这种方式没有实际测试

# 下载安装源
wget http://www.keepalived.org/software/keepalived-1.3.2.tar.gz

# 解压
[root@localhost src]# tar -zvxf keepalived-1.3.2.tar.gz 
[root@localhost src]# cd keepalived-1.3.2

# 配置
[root@localhost keepalived-1.3.2]# ./configure 

# 编译
[root@localhost keepalived-1.3.2]# make && make install

# 复制一些文件
[root@localhost keepalived-1.3.2]# cp /usr/local/src/keepalived-1.3.2/keepalived/etc/init.d/keepalived /etc/rc.d/init.d/
[root@localhost keepalived-1.3.2]# cp /usr/local/etc/sysconfig/keepalived /etc/sysconfig/

# 配置文件
[root@localhost keepalived-1.3.2]# mkdir /etc/keepalived
[root@localhost keepalived-1.3.2]# cp /usr/local/etc/keepalived/keepalived.conf /etc/keepalived/
[root@localhost keepalived-1.3.2]# cp /usr/local/sbin/keepalived /usr/sbin/
```

# 双机热备服务配置

## nginx.conf（主备相同）
&emsp;&emsp;这个是nginx自身用到的配置，主备没有任何区别（因为既然是做主备，那么备的就需要在任何时刻可以随时代替主的，如果有了区别，那就没办法代替了），所以保持一份配置同时更新到主备即可。关于Nginx的配置，因为本篇的主题是环境搭建，所以就不多说了，后面的OpenResty系列会针对Nginx单独进行讲述。
```
user  root;            #运行用户
worker_processes  1;        #启动进程,通常设置成和cpu的数量相等

#全局错误日志及PID文件
error_log  /usr/local/nginx/logs/error.log;
error_log  /usr/local/nginx/logs/error.log  notice;
error_log  /usr/local/nginx/logs/error.log  info;
pid        /usr/local/nginx/logs/nginx.pid;

# 工作模式及连接数上线
events 
{
    use epoll;            #epoll是多路复用IO(I/O Multiplexing)中的一种方式,但是仅用于linux2.6以上内核,可以大大提高nginx的性能
    worker_connections  1024;    #单个后台worker process进程的最大并发链接数
}

#设定http服务器,利用它的反向代理功能提供负载均衡支持
http 
{
    include       mime.types;
    default_type  application/octet-stream;

    #设定请求缓冲
    server_names_hash_bucket_size  128;
    client_header_buffer_size   32K;
    large_client_header_buffers  4 32k;
    # client_max_body_size   8m;
    
    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile        on;
    tcp_nopush     on;
    tcp_nodelay    on;

    #连接超时时间
    keepalive_timeout  65;

    #开启gzip压缩，降低传输流量
    gzip  on;
    gzip_min_length    1k;
    gzip_buffers    4 16k;
    gzip_http_version  1.1;
    gzip_comp_level  2;
    gzip_types  text/plain application/x-javascript text/css  application/xml;
    gzip_vary on;

    #添加tomcat列表，真实应用服务器都放在这
    upstream tomcat_pool 
    {
        #server tomcat地址:端口号 weight表示权值，权值越大，被分配的几率越大;
        server 192.168.1.9:8080 weight=4 max_fails=2 fail_timeout=30s;
    }

    server 
    {
        listen       80;        #监听端口    
        server_name  localhost;
    
        #默认请求设置
        location / {
            proxy_pass http://tomcat_pool;    #转向tomcat处理
        }
        
        #所有的jsp页面均由tomcat处理
        location ~ \.(jsp|jspx|dp)?$
        {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_pass http://tomcat_pool;    #转向tomcat处理
        }

        #定义错误提示页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

## keepalived.conf（主）
&emsp;&emsp;既然nginx是同等的，那么主备的区分就由keepalived来承担了，有主备那肯定需要在配置中有所体现，这里的state就是用来配置主备状态的。
```
global_defs {
    notification_email {
        xxx@qq.com
    }
    notification_email_from sns-lvs@gmail.com
    smtp_server smtp.hysec.com
    smtp_connection_timeout 30
    router_id nginx_master        # 设置nginx master的id，在一个网络应该是唯一的
}
vrrp_script chk_http_port {
    script "/usr/local/src/check_nginx_pid.sh"    #最后手动执行下此脚本，以确保此脚本能够正常执行
    interval 2                          #（检测脚本执行的间隔，单位是秒）
    weight 2
}
vrrp_instance VI_1 {
    state MASTER            # 指定keepalived的角色，MASTER为主，BACKUP为备
    interface eth1            # 当前进行vrrp通讯的网络接口卡(当前centos的网卡)
    virtual_router_id 66        # 虚拟路由编号，主从要一直
    priority 100            # 优先级，数值越大，获取处理请求的优先级越高
    advert_int 1            # 检查间隔，默认为1s(vrrp组播周期秒数)
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
    chk_http_port            #（调用检测脚本）
    }
    virtual_ipaddress {
        192.168.1.200            # 定义虚拟ip(VIP)，可多设，每行一个
    }
}
```

## keepalived.conf（备）
&emsp;&emsp;备机的配置除了state和主不同之外，其余的地方保持和主机一致即可。
```
global_defs {
    notification_email {
        xxx@qq.com
    }
    notification_email_from sns-lvs@gmail.com
    smtp_server smtp.hysec.com
    smtp_connection_timeout 30
    router_id nginx_backup              # 设置nginx backup的id，在一个网络应该是唯一的
}
vrrp_script chk_http_port {
    script "/usr/local/src/check_nginx_pid.sh"
    interval 2                          #（检测脚本执行的间隔）
    weight 2
}
vrrp_instance VI_1 {
    state BACKUP                        # 指定keepalived的角色，MASTER为主，BACKUP为备
    interface eth2                      # 当前进行vrrp通讯的网络接口卡(当前centos的网卡)
    virtual_router_id 66                # 虚拟路由编号，主从要一直
    priority 99                         # 优先级，数值越大，获取处理请求的优先级越高
    advert_int 1                        # 检查间隔，默认为1s(vrrp组播周期秒数)
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_http_port                   #（调用检测脚本）
    }
    virtual_ipaddress {
        192.168.1.200                   # 定义虚拟ip(VIP)，可多设，每行一个
    }
}
```

## check_nginx_pid.sh（主备）
&emsp;&emsp;keepalived是如何检测服务有没有正常运行呢？实际上，跟我们人工常规的检测是一样的，它也是通过检测线程是否存在来判断服务的可用性的，而这个检测脚本就需要我们自行提供并把脚本路径配置到keepalived.conf的script节点中去；当然不能遗忘的一点就是脚本的执行权限必须要保证。
```
[root@localhost keepalived]# cat /usr/local/src/check_nginx_pid.sh 
#!/bin/bash
A=`ps -C nginx --no-header |wc -l`        
if [ $A -eq 0 ];then                            
      /usr/local/nginx/sbin/nginx                #重启nginx
      if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then    #nginx重启失败，则停掉keepalived服务，进行VIP转移
              killall keepalived                    
      fi
fi
[root@localhost keepalived]# 
```

# 主备切换验证

## 主备双活

### keepalived启动日志（主）
```
[root@localhost log]# tail -f messages
Sep  8 11:40:12 localhost Keepalived[3324]: Starting Keepalived v1.2.13 (03/19,2015)
Sep  8 11:40:12 localhost Keepalived[3325]: Starting Healthcheck child process, pid=3327
Sep  8 11:40:12 localhost Keepalived[3325]: Starting VRRP child process, pid=3328
Sep  8 11:40:12 localhost Keepalived_vrrp[3328]: Netlink reflector reports IP 192.168.1.166 added
Sep  8 11:40:12 localhost Keepalived_vrrp[3328]: Netlink reflector reports IP fe80::a00:27ff:fe81:bc35 added
Sep  8 11:40:12 localhost Keepalived_vrrp[3328]: Registering Kernel netlink reflector
Sep  8 11:40:12 localhost Keepalived_vrrp[3328]: Registering Kernel netlink command channel
Sep  8 11:40:12 localhost Keepalived_vrrp[3328]: Registering gratuitous ARP shared channel
Sep  8 11:40:12 localhost Keepalived_vrrp[3328]: Opening file '/etc/keepalived/keepalived.conf'.
Sep  8 11:40:12 localhost Keepalived_healthcheckers[3327]: Netlink reflector reports IP 192.168.1.166 added
Sep  8 11:40:12 localhost Keepalived_healthcheckers[3327]: Netlink reflector reports IP fe80::a00:27ff:fe81:bc35 added
Sep  8 11:40:12 localhost Keepalived_healthcheckers[3327]: Registering Kernel netlink reflector
Sep  8 11:40:12 localhost Keepalived_healthcheckers[3327]: Registering Kernel netlink command channel
Sep  8 11:40:12 localhost Keepalived_healthcheckers[3327]: Opening file '/etc/keepalived/keepalived.conf'.
Sep  8 11:40:12 localhost Keepalived_vrrp[3328]: Configuration is using : 65231 Bytes
Sep  8 11:40:12 localhost Keepalived_vrrp[3328]: Using LinkWatch kernel netlink reflector...
Sep  8 11:40:12 localhost Keepalived_healthcheckers[3327]: Configuration is using : 7526 Bytes
Sep  8 11:40:12 localhost Keepalived_vrrp[3328]: VRRP sockpool: [ifindex(2), proto(112), unicast(0), fd(10,11)]
Sep  8 11:40:12 localhost Keepalived_healthcheckers[3327]: Using LinkWatch kernel netlink reflector...
Sep  8 11:40:13 localhost Keepalived_vrrp[3328]: VRRP_Instance(VI_1) Transition to MASTER STATE
Sep  8 11:40:14 localhost Keepalived_vrrp[3328]: VRRP_Instance(VI_1) Entering MASTER STATE
Sep  8 11:40:14 localhost Keepalived_vrrp[3328]: VRRP_Instance(VI_1) setting protocol VIPs.
Sep  8 11:40:14 localhost Keepalived_vrrp[3328]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth1 for 192.168.1.200
Sep  8 11:40:14 localhost Keepalived_healthcheckers[3327]: Netlink reflector reports IP 192.168.1.200 added
Sep  8 11:40:15 localhost ntpd[1407]: Listen normally on 8 eth1 192.168.1.200 UDP 123
Sep  8 11:40:15 localhost ntpd[1407]: peers refreshed
Sep  8 11:40:19 localhost Keepalived_vrrp[3328]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth1 for 192.168.1.200
```

### keepalived启动日志（备）
```
[root@localhost log]# tail -f messages
Sep  8 11:40:18 localhost Keepalived[3255]: Starting Keepalived v1.2.13 (03/19,2015)
Sep  8 11:40:18 localhost Keepalived[3256]: Starting Healthcheck child process, pid=3258
Sep  8 11:40:18 localhost Keepalived[3256]: Starting VRRP child process, pid=3259
Sep  8 11:40:18 localhost Keepalived_vrrp[3259]: Netlink reflector reports IP 192.168.1.173 added
Sep  8 11:40:18 localhost Keepalived_vrrp[3259]: Netlink reflector reports IP fe80::a00:27ff:fe9a:ad90 added
Sep  8 11:40:18 localhost Keepalived_vrrp[3259]: Registering Kernel netlink reflector
Sep  8 11:40:18 localhost Keepalived_vrrp[3259]: Registering Kernel netlink command channel
Sep  8 11:40:18 localhost Keepalived_vrrp[3259]: Registering gratuitous ARP shared channel
Sep  8 11:40:18 localhost Keepalived_healthcheckers[3258]: Netlink reflector reports IP 192.168.1.173 added
Sep  8 11:40:18 localhost Keepalived_vrrp[3259]: Opening file '/etc/keepalived/keepalived.conf'.
Sep  8 11:40:18 localhost Keepalived_healthcheckers[3258]: Netlink reflector reports IP fe80::a00:27ff:fe9a:ad90 added
Sep  8 11:40:18 localhost Keepalived_healthcheckers[3258]: Registering Kernel netlink reflector
Sep  8 11:40:18 localhost Keepalived_healthcheckers[3258]: Registering Kernel netlink command channel
Sep  8 11:40:18 localhost Keepalived_vrrp[3259]: Configuration is using : 65229 Bytes
Sep  8 11:40:18 localhost Keepalived_vrrp[3259]: Using LinkWatch kernel netlink reflector...
Sep  8 11:40:18 localhost Keepalived_healthcheckers[3258]: Opening file '/etc/keepalived/keepalived.conf'.
Sep  8 11:40:18 localhost Keepalived_healthcheckers[3258]: Configuration is using : 7524 Bytes
Sep  8 11:40:18 localhost Keepalived_vrrp[3259]: VRRP_Instance(VI_1) Entering BACKUP STATE
Sep  8 11:40:18 localhost Keepalived_vrrp[3259]: VRRP sockpool: [ifindex(2), proto(112), unicast(0), fd(10,11)]
Sep  8 11:40:18 localhost Keepalived_healthcheckers[3258]: Using LinkWatch kernel netlink reflector...
```

### 验证服务
&emsp;&emsp;直接通过keepalived配置的虚拟IP来访问即可实现请求的跳转。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-08-02.jpg" width="75%">

## 主死备活

### 手动停止keepalived服务（主）
```
[root@localhost keepalived]# service keepalived stop
停止 keepalived：                                          [确定]
[root@localhost keepalived]# 

```

### keepalived状态（主->死）
```
Sep  8 11:43:58 localhost Keepalived[3325]: Stopping Keepalived v1.2.13 (03/19,2015)
Sep  8 11:43:58 localhost Keepalived_vrrp[3328]: VRRP_Instance(VI_1) sending 0 priority
Sep  8 11:43:58 localhost Keepalived_vrrp[3328]: VRRP_Instance(VI_1) removing protocol VIPs.
Sep  8 11:43:58 localhost Keepalived_healthcheckers[3327]: Netlink reflector reports IP 192.168.1.200 removed
Sep  8 11:43:59 localhost ntpd[1407]: Deleting interface #8 eth1, 192.168.1.200#123, interface stats: received=0, sent=0, dropped=0, active_time=224 secs
Sep  8 11:43:59 localhost ntpd[1407]: peers refreshed
```

### keepalived状态（备->主）
```
Sep  8 11:43:58 localhost Keepalived_vrrp[3259]: VRRP_Instance(VI_1) Transition to MASTER STATE
Sep  8 11:43:59 localhost Keepalived_vrrp[3259]: VRRP_Instance(VI_1) Entering MASTER STATE
Sep  8 11:43:59 localhost Keepalived_vrrp[3259]: VRRP_Instance(VI_1) setting protocol VIPs.
Sep  8 11:43:59 localhost Keepalived_vrrp[3259]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth2 for 192.168.1.200
Sep  8 11:43:59 localhost Keepalived_healthcheckers[3258]: Netlink reflector reports IP 192.168.1.200 added
Sep  8 11:44:01 localhost ntpd[1462]: Listen normally on 7 eth2 192.168.1.200 UDP 123
Sep  8 11:44:01 localhost ntpd[1462]: peers refreshed
Sep  8 11:44:04 localhost Keepalived_vrrp[3259]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth2 for 192.168.1.200
```

### 验证服务
&emsp;&emsp;主挂掉后，通过虚拟IP仍然可以正常实现服务访问。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-08-02.jpg" width="75%">

## 主备双活

### 手动启动keepalived服务（主）
```
[root@localhost keepalived]# service keepalived start
正在启动 keepalived：                                      [确定]
[root@localhost keepalived]# 
```

### keepalived状态（主->挂->主）
```
Sep  8 11:46:50 localhost Keepalived[3589]: Starting Keepalived v1.2.13 (03/19,2015)
Sep  8 11:46:50 localhost Keepalived[3590]: Starting Healthcheck child process, pid=3592
Sep  8 11:46:50 localhost Keepalived[3590]: Starting VRRP child process, pid=3593
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: Netlink reflector reports IP 192.168.1.166 added
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: Netlink reflector reports IP fe80::a00:27ff:fe81:bc35 added
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: Registering Kernel netlink reflector
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: Registering Kernel netlink command channel
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: Registering gratuitous ARP shared channel
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: Opening file '/etc/keepalived/keepalived.conf'.
Sep  8 11:46:50 localhost Keepalived_healthcheckers[3592]: Netlink reflector reports IP 192.168.1.166 added
Sep  8 11:46:50 localhost Keepalived_healthcheckers[3592]: Netlink reflector reports IP fe80::a00:27ff:fe81:bc35 added
Sep  8 11:46:50 localhost Keepalived_healthcheckers[3592]: Registering Kernel netlink reflector
Sep  8 11:46:50 localhost Keepalived_healthcheckers[3592]: Registering Kernel netlink command channel
Sep  8 11:46:50 localhost Keepalived_healthcheckers[3592]: Opening file '/etc/keepalived/keepalived.conf'.
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: Configuration is using : 65231 Bytes
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: Using LinkWatch kernel netlink reflector...
Sep  8 11:46:50 localhost Keepalived_healthcheckers[3592]: Configuration is using : 7526 Bytes
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: VRRP sockpool: [ifindex(2), proto(112), unicast(0), fd(10,11)]
Sep  8 11:46:50 localhost Keepalived_healthcheckers[3592]: Using LinkWatch kernel netlink reflector...
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: VRRP_Instance(VI_1) Transition to MASTER STATE
Sep  8 11:46:50 localhost Keepalived_vrrp[3593]: VRRP_Instance(VI_1) Received lower prio advert, forcing new election
Sep  8 11:46:51 localhost Keepalived_vrrp[3593]: VRRP_Instance(VI_1) Entering MASTER STATE
Sep  8 11:46:51 localhost Keepalived_vrrp[3593]: VRRP_Instance(VI_1) setting protocol VIPs.
Sep  8 11:46:51 localhost Keepalived_vrrp[3593]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth1 for 192.168.1.200
Sep  8 11:46:51 localhost Keepalived_healthcheckers[3592]: Netlink reflector reports IP 192.168.1.200 added
Sep  8 11:46:52 localhost ntpd[1407]: Listen normally on 9 eth1 192.168.1.200 UDP 123
Sep  8 11:46:52 localhost ntpd[1407]: peers refreshed
Sep  8 11:46:56 localhost Keepalived_vrrp[3593]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth1 for 192.168.1.200
```

### keepalived状态（备->主->备）
```
Sep  8 11:46:50 localhost Keepalived_vrrp[3259]: VRRP_Instance(VI_1) Received higher prio advert
Sep  8 11:46:50 localhost Keepalived_vrrp[3259]: VRRP_Instance(VI_1) Entering BACKUP STATE
Sep  8 11:46:50 localhost Keepalived_vrrp[3259]: VRRP_Instance(VI_1) removing protocol VIPs.
Sep  8 11:46:50 localhost Keepalived_healthcheckers[3258]: Netlink reflector reports IP 192.168.1.200 removed
Sep  8 11:46:52 localhost ntpd[1462]: Deleting interface #7 eth2, 192.168.1.200#123, interface stats: received=0, sent=0, dropped=0, active_time=171 secs
Sep  8 11:46:52 localhost ntpd[1462]: peers refreshed
```

### 验证服务
&emsp;&emsp;主恢复后，通过虚拟IP仍旧可以正常实现访问。
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-09-08-02.jpg" width="75%">

