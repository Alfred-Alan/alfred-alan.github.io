---
layout: post
title: 'Docker的一些常用操作'
description: 'Docker的常用操作'
categories: [Docker]
image: /assets/img/blog/docker.png
related_posts:
  - _posts/2020-7-15-docker-install-mysql.md
  - _posts/2020-7-20-use-docker-deploy.md
---
- Table of Contents
{:toc .large-only}


## 镜像命令：

###  显示docker 版本 
```powershell
docker version
```

### 显示docker 系统信息
```powershell
docker info
```

### 下载镜像
可以到[docker_hub](https://cloud.docker.com/)来获取镜像 方便使用
例：
```powershell
docker pull redis:latest
```


### 查看镜像
```powershell
docker images
```
```powershell
$ docker  images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
镜像的仓库源	 镜像的标签	        镜像ID             镜像创建时间         镜像大小
```

### 搜索镜像
```powershell
docker search 镜像名字
```

### 导入镜像
```powershell
docker load -i '镜像路径'
```

### 导出镜像
```powershell
docker save -o '路径' 镜像名称
```

### 删除镜像：

- 删除单个： docker rmi -f 镜像ID
- 删除多个： docker rmi -f 镜像名1:TAG 镜像名2:TAG
- 删除全部： docker rmi -f $(docker images -qa)

## 容器命令：

### 查看所有容器
```powershell
docker ps -a
```

### 启动容器
```powershell
docker run --name myredis -d 6379:6379 redis
```

### 进入容器
```powershell
docker exec -it 容器名称/ID bash
```

### 停止容器
```powershell
docker stop 容器名称/ID
```

### 删除容器

```powershell
docker rm 容器ID
```

一次性删除多个容器

```powershell
docker rm 容器ID 容器ID
```

删除所有

```powershell
docker rm -f $(docker ps -a -q)
```

