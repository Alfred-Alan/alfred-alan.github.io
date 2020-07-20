---
layout: post
title: '利用DockerHub部署Nginx反向代理Gunicorn+Flask独立架构'
subtitle: '如何利用DockerHub部署Nginx反向代理Gunicorn+Flask独立架构'
date: 2020-07-21
categories: 技术
tags: docker
image: /assets/img/blog/docker.png
---

转载为 https://v3u.cn/a_id_165

本次使用Nginx反向代理Flask服务，为什么要加一层Nginx呢？因为Nginx可以直接处理静态文件请求而不用经过应用服务器，避免占用宝贵的运算资源，并且可以缓存静态资源，使访问静态资源的速度有效提高。同时它可以吸收一些瞬时的高并发请求，让Nginx先保持住连接(缓存http请求)，然后后端慢慢消化掉这些并发。当然了，最重要的一点就是Nginx可以提供负载均衡策略，这样我们的应用服务就可以横向扩展，分担压力了。

  大体架构如下：

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717130740_58940.png)

  首先出场的是贵为Docker三大核心之一的DockerHub(仓库)，我们可以将打包好的镜像免费push到上面，就这样就可以随时pull自己的镜像，注册地址:https://hub.docker.com/

  激活账号以后，创建仓库，这一步和github创建代码仓库差不太多。

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717130718_29278.png)

  填写仓库信息具体为仓库名称、描述以及是否公开或者私有。

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717130715_73780.png)

  创建成功之后，它就会出现在镜像列表中

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717130741_50187.png)

  此时我们需要对本地的镜像重命名，这里重命名为zcxey2911/myflask。因为要与dockerhub上的仓库对应。如果名称不对应是无法将本地镜像push到线上仓库中。

```powershell
docker tag myflask zcxey2911/myflask
```

 ![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717140724_55288.png)

  之后在命令行输入命令

```powershell
docker login
```

  用DockerHub的账号和密码登录

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717140746_73706.png)

  登录成功之后，用命令把本地镜像push到hub中

```powershell
docker push zcxey2911/myflask
```

  注意这里的镜像名称必须和hub中的仓库名称一致，否则将会抛出错误。

  上传成功后，就可以在DockerHub中看到它了，此时就能随意pull操作了

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717140726_83859.png)

  前置操作已经完毕，此时，登录你的云服务器，这里以百度云的Centos7.7为例子，进入服务器后安装Docker服务

```powershell
#升级yum
sudo yum update
#卸载旧版本docker
sudo yum remove docker  docker-common docker-selinux docker-engine
#安装依赖
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
#设置源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum makecache fast
#安装docker
sudo yum install docker-ce

#启动服务
sudo systemctl start docker
```

  安装完成后键入 docker -v

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717140725_25251.png)

  返回Docker版本号说明没有问题。

  拉取我们之前打包并且上传到hub的Flask镜像

```powershell
docker pull zcxey2911/myflask
```

  下载成功后，会展示在镜像库里

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717140744_90909.png)

  运行项目，这里我们可以采用后台守护进程的模式起服务

```powershell
sudo docker run -d -p 5000:5000 --name testflask zcxey2911/myflask
```

  使用docker ps命令可以看到是否运行成功。

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717140722_36527.png)

  使用服务器的ip访问一下Flask服务，这里有个小坑，不论是腾讯云、阿里云还是百度云亦或是各种乱七八糟的云，都需要在安全组策略中开放你需要访问的端口，比如这里我用的5000。

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717140715_76493.png)

  好了，现在我们同样利用Docker来安装Nginx服务

```powershell
docker pull nginx
```

  随后启动Nginx测试一下

```powershell
docker run -d -p 80:80 nginx
```

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717140744_66365.png)

  现在，我们将运行Nginx容器里的配置文件copy到宿主机里面

  前面是容器的路径 后面是宿主机的路径

```powershell
docker cp 容器id:/etc/nginx/conf.d/default.conf /root/default.conf
```

  容器id可以通过docker ps命令查看

  复制出来之后，输入命令修改这个nginx配置

```powershell
vim /root/default.conf
```

  将Gunicorn配置加到里面

```powershell
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

     location / {
        proxy_pass http://你的服务器公网ip:5000; # 这里是指向 gunicorn host 的服务地址
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
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

  修改完配置文件之后，关掉运行的nginx服务容器，并且删掉它

```powershell
docker stop 容器iddocker rm $(docker ps -a -q)
```

  随后再次启动Nginx容器，不过这次和上次不同之处就是需要用到 -v 进行挂载了，挂载简单理解就是将宿主机的文件替换Docker容器内部的文件，达到修改的效果。

```powershell
docker run --name mynginx -d -p 80:80 -v /root/default.conf:/etc/nginx/conf.d/default.conf nginx
```

  这里-v参数也遵循冒号左侧为宿主机右侧为容器的原则。

  重新启动成功后，访问服务器ip

![img](https://v3u.cn/v3u/Public/js/editor/attached/20200717150741_75778.png)

  发现已经部署成功，整个流程轻松加愉快，比原始的命令行shell安装不知快了多少倍，最后奉上Dockerhub地址：https://hub.docker.com/r/zcxey2911/myflask 和 Flask工程地址:https://gitee.com/QiHanXiBei/myflask