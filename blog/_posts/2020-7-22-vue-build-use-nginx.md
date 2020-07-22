---
layout: post
title: '使用docker部署Nginx反向代理Vue'
subtitle: '使用docker部署Nginx反向代理Vue'
date: 2020-07-22
categories: 技术
tags: docker vue nginx 
image: /assets/img/blog/vuejs-logo.jpg

---

在我们编写好项目之后 ，我们需要将Vue项目打包上线

常见的方法就是使用git 将项目上传到 服务器中 

然后在服务器中启动该项目，而此类方法有一个弊端，无法一键启动&多个项目同时部署 

而我使用的方法是 使用 nginx代理 vue，然后在打包为 docker 镜像 
使用docker进行统一部署 大大的降低了项目的耦合性 


首先在开发完的项目内输入以下命令

```powershell 
npm run build
```

执行完成之后会 出现一个dist文件夹, 里面就是根据vue配置 生成的静态文件 

![dist](/assets/img/vuebuild/dist.png)

<br>

此时需要注意如果在index 中导入了项目内的样式

我之前遇到的坑是在上线之后样式不会渲染 找不到路径

需要把地址 从 ``./`` 改为 ``/``

![更改css 路径](/assets/img/vuebuild/更改css 路径.png)

<br>

然后在根目录下创建 nginx 文件夹 
在里面创建一个 default.conf 配置文件 

![创建nginx文件夹](/assets/img/vuebuild/创建nginx文件夹.png)

<br>

该文件是指定nginx的配置 
将下列配置放进配置文件中

```powershell
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    access_log  /var/log/nginx/host.access.log  main;
    error_log  /var/log/nginx/error.log  error;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

　 注意⚠️：如果vue-router使用的是history模式，try_files $uri $uri/ /index.html; 非常重要！！！

　　因为我们的应用是单页客户端应用，如果后台没有正确的配置，当用户在浏览器访问地址时，就会返回404。

　　所以需要在服务端增加一个覆盖所有情况的候选资源，如果URL匹配不到任何静态资源，则应该返回同一个index.html页面，这个页面就是你app依赖的页面。

　　上面的文件定义了首页的指向为 `/usr/share/nginx/html/index.html`, 所以我们可以一会把构建出来的index.html文件和相关的静态资源放到`/usr/share/nginx/html`目录下。



nginx 配置好之后 

在根目录下还要创建一个文件 为 ``Dockerfile``

![创建dockerfile](/assets/img/vuebuild/创建dockerfile.png)

<br>

该文件被docker 读取的时候 docker会按内部的设置来生成镜像

```powershell
# 设置基础镜像
FROM nginx
# 定义作者
MAINTAINER L
# 将dist文件中的内容复制到 /usr/share/nginx/html/ 这个目录下面
COPY dist/  /usr/share/nginx/html/
#用本地的 default.conf 配置来替换nginx镜像里的默认配置
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
```

当配置设置完毕之后 
就可以使用docker对项目进行打包，打包为一个独立的镜像 
docker build -t 'myvue' .

```powershell
[root@VM-0-12-centos vue_project]# docker build -t 'myvue' .
Sending build context to Docker daemon 11.39 MB
Step 1/4 : FROM nginx
 ---> 0901fa9da894
Step 2/4 : MAINTAINER bywlop
 ---> Running in 29aeca4a4375
 ---> 6d97547f1eaf
Removing intermediate container 29aeca4a4375
Step 3/4 : COPY dist/ /usr/share/nginx/html/
 ---> 9013a6349886
Removing intermediate container f208da27c49a
Step 4/4 : COPY nginx/default.conf /etc/nginx/conf.d/default.conf
 ---> 17537a1a2f41
Removing intermediate container 5b56bb5d8202
Successfully built 17537a1a2f41
```

因为打包时候会下载vue 执行的所有环境
需要等待一些时间 

打包完成之后 
我们输入 docker images 就可以看到项目已经在镜像中了 

```powershell
[root@VM-0-12-centos ~]# docker images 
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
myvue                 latest              17537a1a2f41        4 hours ago         139 MB
docker.io/nginx        latest              0901fa9da894        11 days ago         132 MB
docker.io/python       3.6                 e0373ff33a19        3 weeks ago         914 MB
```

最后使用docker 启动该镜像 
docker run -d -p 80:80 myvue 

查看是否启动

```powershell
[root@VM-0-12-centos ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   
b7b7362eb14c        myvue              "/docker-entrypoin..."   4 hours ago         Up 4 hours          0.0.0.0:80->80/tcp       

```

