---
layout: post
title: 'windows 安装多版本nodejs'
description: 'Windows下使用nvm维护多版本nodejs'
categories: [Windows, node]
tags: []
image:  /assets/img/blog/windows-install-nvm-.png

related_posts:
  - 
---

- Table of Contents
{:toc .large-only}

### 安装nvm
废话不多说

直接去github找一个windwos版本的安装包

https://github.com/coreybutler/nvm-windows/releases

找到exe 下载到本地

![image-20230113103623943](/assets/2022-11-24-nvm-install.assets/image-20230113103623943.png)

点击安装包， 我这里设置的安装目录是 `E:\Program Files\`

后面所有通过nvm下载的node源文件都会在这个路径里面

![image-20230113103833784](/assets/2022-11-24-nvm-install.assets/image-20230113103833784.png)

然后设置nodejs软连接路径，nvm选择node版本后，会通过镜像连接形式存在这个文件夹内

这样就形成了版本号的无缝切换

![image-20230113104156947](/assets/2022-11-24-nvm-install.assets/image-20230113104156947.png)


### 配置镜像源

我们在`cmd|powerhsell`内输入以下两条命令

设置node 和 npm 下载的镜像源

```powershell
nvm node_mirror https://npm.taobao.org/mirrors/node/
nvm npm_mirror https://npm.taobao.org/mirrors/npm/
```

### 安装卸载版本
```powershell
nvm install 14.19.1
nvm uninstall 14.19.1
```
这里我一共下载了两个版本

```powershell
C:\Users\admin>nvm install 12.22.12
Downloading node.js version 12.22.12 (64-bit)...
Complete
Creating E:\Program Files\nvm\temp

Downloading npm version 6.14.16... Complete
Installing npm v6.14.16...

Installation complete. If you want to use this version, type

nvm use 12.22.12

C:\Users\admin>nvm install 14.19.1
Downloading node.js version 14.19.1 (64-bit)...
Complete
Downloading npm version 6.14.16... Complete
Installing npm v6.14.16...

Installation complete. If you want to use this version, type

nvm use 14.19.1

```

### 查看所有版本

```powershell
nvm list
```

```powershell
C:\Windows\system32> nvm list

    14.19.1
    12.22.12
```

### 使用版本

```powershell
nvm use 版本号
```
注意切换版本后，npm 下载包的镜像源需要重新设置
{:.note title="Attention"}
可以看见在当前使用版本号前加`*`
```powershell
C:\Windows\system32> nvm use 12.22.12
Now using node v12.22.12 (64-bit)

C:\Windows\system32> nvm list

    14.19.1
  * 12.22.12 (Currently using 64-bit executable)
```


