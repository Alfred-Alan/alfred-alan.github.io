---
layout: post
title: 'Windows安装Docker'
description: '如何在windwos上安装docker'
categories: [Docker]
image: /assets/img/blog/windows_docker.png
related_posts:
  - _posts/2020-04-27-docker-use.md
  - _posts/2020-7-15-docker-install-mysql.md

---
- Table of Contents
{:toc .large-only}

## Docker是什么

Docker 是一个开源的应用容器引擎，基于 [Go 语言](https://www.runoob.com/go/go-tutorial.html) 并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

## Docker的应用场景

- Web 应用的自动化打包和发布。

- 自动化测试和持续集成、发布。

- 在服务型环境中部署和调整数据库或其他的后台应用。

- 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境

## Docker的安装

win7、win8 等需要利用 docker toolbox 来安装 [安装地址](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/)

下载后安装

![安装界面](/assets/img/docker/install_view.png)

如果安装了git的话就不要选

![git](/assets/img/docker/git.png)

只选上面两个

![选俩](/assets/img/docker/select_option.png)

之后一直next 安装

在安装完成后

![修改目录](/assets/img/docker/change_address.png)

第一次启动后会在C://user 目录下生成对应的文件夹

我们先关闭docker窗口 替换文件

![更改文件](/assets/img/docker/change_file.png)

之后断网在打开docker  不然docker会检测下载 boot2docker.io 

启动完成后会出现下面界面

![docker界面](/assets/img/docker/docker_home.png)

之后到阿里云去申请一个镜像加速器

![镜像源](/assets/img/docker/mirror_source.png)

在docker界面输入

```dos
# 以此加速地址创建虚拟机
docker-machine create --engine-registry-mirror=加速地址 -d virtualbox default
# 退出
exit 
# 重启虚拟机
docker-machine restart default
```

之后输入`docker info`查看是否配置好

![检测](/assets/img/docker/check.png)
