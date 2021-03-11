---
layout: post
title: '在阿里云Centos上配置nginx负载均衡配置'
description: '如何在阿里云Centos上配置nginx负载均衡配置'
categories: [Docker, Nginx]

image: /assets/img/blog/nginx.png
accent_image: /assets/img/blog/nginx.png
# invert_sidebar: true
---

- Table of Contents
{:toc .large-only}

## 前言
通常在一些访问量 极大的情况下 
我们的服务器可能没有那么强悍的性能来支持高并发高负载
这时候就需要用到Nginx的负载均衡 

## Nginx
Nginx是一个高性能的HTTP和反向代理的服务器 由于Nginx的高性能和稳定性使得国内使用 Nginx 作为 Web 服务器的网站也越来越多，其中包括新浪、网易、腾讯、搜狐等企业的一些门户网站等



## upstream常用参数解析 
down：此服务器不接受请求

weight：默认为1，wight越大，负载的权重越大。

max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误。

fail_timeout：max_fails此失败后，暂停的时间。

backup：其他所有非backup的机器宕掉或者忙的时候，请求backup的机器。


## 负载均衡算法


**轮询**：将请求按顺序轮流地分配到后端服务器上，它均衡地对待后端的每一台服务器，而不关心服务器实际的连接数和当前的系统负载。

```powershell
upstream mServer {
    server 39.96.70.17:5000 max_fails=2 fail_timeout=10s; 
    server 139.155.72.14:5000 max_fails=2 fail_timeout=10s; 
}
```
<br>

**加权轮询**：不同的后端服务器可能机器的配置和当前系统的负载并不相同，因此它们的抗压能力也不相同。
           给配置高、负载低的机器配置更高的权重，让其处理更多的请；而配置低、负载高的机器，给其分配较低的权重，
           降低其系统负载，加权轮询能很好地处理这一问题，并将请求顺序且按照权重分配到后端。
           weight 默认为1，weight越大，负载的权重就越大
```powershell
upstream mServer {
    server 39.96.70.17:5000 weight=9 max_fails=2 fail_timeout=10s; 
    server 139.155.72.14:5000 weight=10 max_fails=2 fail_timeout=10s; 
}
```

<br>

**IP哈希**：根据获取客户端的IP地址，通过哈希函数计算得到一个数值，用该数值对服务器列表的大小进行取模运算，
                  得到的结果便是客服端要访问服务器的序号。采用源地址哈希法进行负载均衡，
                  同一IP地址的客户端，当后端服务器列表不变时，它每次都会映射到同一台后端服务器进行访问。
```powershell
upstream mServer {
    ip_hash;
    server 39.96.70.17:5000 max_fails=2 fail_timeout=10s; 
    server 139.155.72.14:5000 max_fails=2 fail_timeout=10s; 
}
```
<br>

**最小连接数**：由于后端服务器的配置不尽相同，对于请求的处理有快有慢，最小连接数法根据后端服务器当前的连接情况，
            动态地选取其中当前积压连接数最少的一台服务器来处理当前的请求，
            尽可能地提高后端服务的利用效率，将负责合理地分流到每一台服务器。
```powershell
upstream mServer {
    least_conn;
    server 39.96.70.17:5000 max_fails=2 fail_timeout=10s; 
    server 139.155.72.14:5000 max_fails=2 fail_timeout=10s; 
}
```
**优先相应**：fair算法按后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身不支持fair，
            如果需要这种调度算法，则必须安装upstream_fair模块。

```powershell
upstream mServer {  
    server 39.96.70.17:5000 max_fails=2 fail_timeout=10s; 
    server 139.155.72.14:5000 max_fails=2 fail_timeout=10s; 
    fair;  
```

## 流程

![Nginx流程](/assets/img/nginx/Nginx_process.png)

## 如何配置负载均衡

首先使用docker 下载nginx 镜像 

```powershell
docker pull nginx
```

随后启动Nginx测试一下

```powershell
docker run -d -p 80:80 nginx
```
## 修改Nginx配置
因为我们需要更改 nginx中的配置 
就要把容器中的配置 复制到主机内 

```powershell
docker cp 容器id:/etc/nginx/conf.d/default.conf /root/default.conf
```

复制出来之后 使用vim 来打开 ``default.conf``

将负载均衡配置添加进去

```shell
# file: '/root/default.conf'
upstream mServer{

        server 39.96.70.17:5000;
        server 139.155.72.14:5000;
}

server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
        proxy_pass http://mServer;
        proxy_set_header Host $host;
    }
    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}	
```

其中 proxy_pass 指向 upstream 的代理组
可以将请求分发到其中的ip中 
从而实现简单的负载均衡 

修改完配置文件之后  关闭正在运行的nginx 的服务器
并且删除它

```powershell
docker stop 容器id
docker rm 容器id
```

## 启动Nginx
随即使用本地配置文件指定启动nginx

```powershell
docker run --name mynginx -d -p 80:80 -v /root/default.conf:/etc/nginx/conf.d/default.conf nginx
```

这里-v参数也遵循冒号左侧为宿主机右侧为容器的原则。

重新启动成功后，访问服务器ip

之后就可以测试 第一条请求回进入第一个ip
而第二条请求就被分发到第二个ip中

