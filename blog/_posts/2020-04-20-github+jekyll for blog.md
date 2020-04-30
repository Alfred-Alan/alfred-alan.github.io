---
layout: post
title: '使用github和jekyll实现静态博客'
subtitle: '如何使用github和jekyll来实现静态博客'
date: 2020-04-20
categories: 技术
tags: ruby jekyll github blog
image: /assets/img/blog/jekyll.png
---




## 如何使用github和jekyll 来实现静态博客

1.注册一个GIthub账号

2.创建一个库 命名为 用户名.github.io

![命名](https://img-blog.csdnimg.cn/20190606135742783.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4MjI1NTU4,size_16,color_FFFFFF,t_70)

3.从jekyll官网找到一个喜欢的模板 <http://jekyllthemes.org/>

![官网](https://img-blog.csdnimg.cn/20190606124423999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4MjI1NTU4,size_16,color_FFFFFF,t_70)

4.将模板保存下来 但是在上传到库的时候一定是根目录 像图片里一样 不要将一整个文件夹上传！！！

或者直接fork到自己创建的库下

![fork](https://img-blog.csdnimg.cn/20190606124353548.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4MjI1NTU4,size_16,color_FFFFFF,t_70)

这样就可以直接打开 http://用户名.github.io 直接访问

5.下载环境

这时候就要用到ruby 

我们需要下载安装以下几个工具

* Ruby
* Rubygems
* jekyll

### 安装Ruby

ruby官网下载安装：<https://rubyinstaller.org/downloads/>  ruby+devkit 2.6.6-1(x64)

安装完成后配置环境变量

在命令提示符中，得到ruby版本号，如下图，即安装成功

```dos
C:\Users\haoyang>ruby -v
ruby 2.1.3p242 <2014-09-19 revision 47630> [i386-mingw321]
```

### 安装RubyGems

官网下载 <http://rubygems.org/pages/download> rubygems-2.4.5.zip

cd到RubyGems目录

```dos
E:\xunlei\rubygems-3.1.2>
```

执行安装

```dos
E:\xunlei\rubygems-3.1.2>ruby setup.rb
```



### 用RubyGems安装Jekyll

在使用gem安装的时候  我遇到了一些问题  原本的源特别慢   我们就需要更改gem下载源

```dos
# gem 删除默认源命令
gem sources --remove https://rubygems.org/

# gem 添加国内源
gem sources --add https://gems.ruby-china.com/
（官网给的原来的淘宝镜像已失效）

检测方法
gem sources -l
如果输出如下，代表添加成功
*** CURRENT SOURCES ***
https://gems.ruby-china.com/
```

更换源之后再执行下面的语句安装

```dos
C:\Windows\system32>gem install jekyll
```

6.在本地编写项目

```dos
jekyll new myblog 生成项目
jekyll build 生成 每次要提交的事时候也需要生成一下
默认会将网站生成到 ./_site 目录下，生成目录可以通过配置文件 ./_config.yml
```

_posts 文件夹用来存储文章  

index.html 展示主页

_config.yml  配置文件

#### 运行项目

当所有都准备好之后使用编辑软件打开_config.yml

在底部添加 port: 4001

使用cmd进入之前下载好的项目内

 输入 jekyll serve 开启服务

这样在启动项目的时候就可以访问 http://127.0.0.1:4001 查看项目状态



将写好的md 文件命名为2017-04-18-hello-jekyll.md

注意 ：有些文件编码可能跟jekyll 不一致 这样就需要更改配置 

一般是在_config.yml 里添加encoding: UTF-8

如果还是解决不了的话 请根据jekyll版本来百度解决 我自己是每个试出来的 误打误撞...

当编写完成后 使用jekyll build打包 在进行提交到指定的库中