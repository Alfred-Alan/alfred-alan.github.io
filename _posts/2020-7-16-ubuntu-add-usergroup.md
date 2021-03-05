---
layout: post
title: 'Ubuntu中输入命令不加sudo'
description: '如何在ubutnu下输入命令不加sudo'
categories: [Ubuntu]
image: /assets/img/blog/Ubuntu.png
accent_image: /assets/img/blog/Ubuntu.png
related_posts:
  - _posts/2020-7-14-ubuntu-install-vim.md
---

- Table of Contents
{:toc .large-only}



## 如何使用额外命令不加sudo
在使用额外命令的时候 不可避免的都需要输入``sudo``

有时候就没必要每次都要输入``sudo ``

我们就把``docker`` 命令添加进用户组

这样每次使用docker就不需要再前面输入``sudo``

教程以下：

## 查看用户组及成员

```powershell
sudo cat /etc/group | grep docker
```

## 添加docker组

```powershell
sudo groupadd docker 
```

## 用户到docker组 

```powershell
sudo gpasswd -a ${USER} docker 
```

## 增加读写权限

```powershell
 sudo chmod a+rw /var/run/docker.sock
```

## 重启docker

```powershell
sudo systemctl restart docker 
```

同理将一些常见的命令也可以添加进 用户组

