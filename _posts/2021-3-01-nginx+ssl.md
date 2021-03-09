---
layout: post
title: 'Nginx配置ssl证书'
description: '如何将Nginx代理的域名升级成https'
categories: [Nginx]
image:  /assets/img/blog/ssl_nginx.png
# accent_image: /assets/img/blog/RabbitMQ-logo.png
# invert_sidebar: true
related_posts:
  - _posts/2021-2-25-jekyll-nginx.md
  - _posts/2020-04-27-docker-use.md
---
- Table of Contents
{:toc .large-only}

## 申请ssl证书

我们要想把http变成https的话 就要先申请ssl 证书
我这里使用的是阿里云的云盾ssl证书



### 点击SSl证书

第一步登录阿里云 点击ssl证书

![select_product](\assets\img\ssl\select_product.png)

### 选购证书

进入到ssl 证书页面 我们点击选购证书

![purchase_certificate](\assets\img\ssl\purchase_certificate.png)

### 云盾证书

点击云盾证书服务 然后 选择DV单域名证书 

因为是一年免费20次 所以这里就白嫖了

![dv_ssl_20](\assets\img\ssl\dv_ssl_20.png)

### 证书控制台

购买完成后 进入证书控制台 点击证书申请

![certificate_request_ssl](\assets\img\ssl\certificate_request_ssl.png)



因为我们只有免费的单域名证书 点击消耗一次



![request_free](\assets\img\ssl\request_free.png)



### 证书信息

我们申请之后 会在证书栏看见刚才申请的证书

![certificate_request](\assets\img\ssl\certificate_request.png)

这个时候的证书申请 就是填写你的信息

然后把域名和身份信息 输入进去

![fill_information](\assets\img\ssl\fill_information.png)

申请之后我们可以选择验证一下

![verification](\assets\img\ssl\verification.png)

然后等待审核完成就可以下载证书配置了

### 下载证书

证书审核完成之后就可以点击下载

![download](\assets\img\ssl\download.png)

我们选择nginx的ssl证书

![download_nginx_ssl](\assets\img\ssl\download_nginx_ssl.png)



下载之后 我们会看见有两个文件

```powershell
xxxxx.key
xxxxx.pem
```

## 配置Nginx

### 进入目录

在服务器上 进入nginx 目录 没有则创建

```shell
cd /home/nginx/
```



### 创建证书文件夹

创建``ssl`` 文件夹

```shell
mkdir ssl
```

将验证好的ssl 证书上传到此文件夹内

### 修改配置文件

```shell
vim /home/nginx/config/conf.d/default.conf
```

```shell
# file: '/home/nginx/config/conf.d/default.conf'
server {
    listen       80;
    listen       443 ssl;
    server_name  localhost;

    # 注意文件位置，是从/etc/nginx/下开始算起的
    # ssl证书地址
    ssl_certificate     /etc/nginx/ssl/ssl.pem;  # pem文件的路径
    ssl_certificate_key /etc/nginx/ssl/ssl.key;  # key文件的路径

    # ssl验证相关配置
    ssl_session_timeout  5m;    #缓存有效期
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;    #加密算法
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;    #安全链接可选的加密协议
    ssl_prefer_server_ciphers on;   #使用服务器端的首选算法

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

```



## docker启动nginx



**启动nginx容器**

创建一个拥有挂载目录的，能够自动重启的Nginx容器

```shell
docker run --name nginx -d -p 80:80 -p 443:443 \

--restart always \

-v /home/nginx/log:/var/log/nginx \

-v /home/nginx/config/conf.d:/etc/nginx/conf.d \

-v /home/nginx/nginx.conf:/etc/nginx/nginx.conf \

-v /root/alfred-alan.github.io/_site:/usr/share/nginx/html \
-v /home/nginx/ssl/:/etc/nginx/ssl/ nginx
```





