---
layout: post
title: '如何安装Docker及Docker的一些操作'
subtitle: '将redis存入docker容器使用'
date: 2020-04-27
categories: 技术
tags: Docker
image: /assets/img/blog/docker.png
---
## 如何安装Docker及Docker的一些操作
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

![安装界面](/assets/img/day7/安装界面.png)

如果安装了git的话就不要选

![git](/assets/img/day7/git.png)

只选上面两个

![选俩](/assets/img/day7/选俩.png)

之后一直next 安装

在安装完成后

![修改目录](/assets/img/day7/修改目录.png)

第一次启动后会在C://user 目录下生成对应的文件夹

我们先关闭docker窗口 替换文件

![更改文件](/assets/img/day7/更改文件.png)

之后断网在打开docker  不然docker会检测下载 boot2docker.io 

启动完成后会出现下面界面

![docker界面](/assets/img/day7/docker界面.png)

之后到阿里云去申请一个镜像加速器

![镜像源](/assets/img/day7/镜像源.png)

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

![检测](/assets/img/day7/检测.png)

#### docker命令

##### 镜像命令：

可以到[docker_hub](https://cloud.docker.com/)来获取镜像 方便使用 例：docker pull redis:latest

显示docker 版本 ：`docker version`

显示docker 系统信息： `docker info`

查看镜像 ：`docker images`

```dos
$ docker  images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
镜像的仓库源		  镜像的标签			  镜像ID			   镜像创建时间		镜像大小
```

搜索镜像：`docker search 镜像名字`

导入镜像： `docker load -i '镜像路径'`

导出镜像： `docker save -o '路径' redis`

删除镜像：

- 删除单个： docker rmi -f 镜像ID
- 删除多个： docker rmi -f 镜像名1:TAG 镜像名2:TAG
- 删除全部： docker rmi -f $(docker images -qa)

##### 容器命令：

查看容器：`docker ps `

根据镜像启动容器 ：`docker run -p 6380:6379redis redis-server`

本机启动docker服务：`redis-cli -h dockerIP -p docker启动的端口`

停止容器 ：`docker stop 容器ID`

删除一个容器

```dos
docker rm 容器ID
```

一次性删除多个容器

```dos
docker rm 容器ID 容器ID
```

删除所有

```dos
docker rm -f $(docker ps -a -q)
```

