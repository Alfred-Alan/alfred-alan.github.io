---
layout: post
title: '使用github和jekyll实现静态博客'
categories: [Ruby]
tags: [jekyll,github]
image: /assets/img/blog/jekyll.png
description: '如何使用github和jekyll 来实现静态博客'

related_posts:
  - _posts/2020-04-20-ruby-install.md
---

- Table of Contents
{:toc .large-only}

1.注册一个GIthub账号

2.创建一个库 命名为 用户名.github.io

![命名](https://img-blog.csdnimg.cn/20190606135742783.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4MjI1NTU4,size_16,color_FFFFFF,t_70)

<br>

3.从jekyll官网找到一个喜欢的模板 <http://jekyllthemes.org/>

![官网](https://img-blog.csdnimg.cn/20190606124423999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4MjI1NTU4,size_16,color_FFFFFF,t_70)

<br>

4.将模板保存下来 但是在上传到库的时候一定是根目录 像图片里一样 不要将一整个文件夹上传！！！

或者直接fork到自己创建的库下

![fork](https://img-blog.csdnimg.cn/20190606124353548.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4MjI1NTU4,size_16,color_FFFFFF,t_70)

这样就可以直接打开 http://用户名.github.io 直接访问

<br>

5.下载环境

这时候就要用到ruby 

我们需要下载安装以下几个工具

* Ruby
* jekyll
* bundler

[Ruby安装](http://alfred-alan.github.io/blog/ruby/2020-04-20-ruby-install/)
{:.note title="移步"}

6.在本地编写项目

创建项目

```powershell
jekyll new my-site
```
安装所需依赖包

bundle 换国内源

```
bundle config mirror.https://rubygems.org https://gems.ruby-china.com
```

这样你不用改你的 Gemfile 的 source。

```powershell
cd my-site
bundle install
```

_posts 文件夹用来存储文章  

index.html 展示主页

_config.yml  配置文件

<br>

## 运行项目

当所有都准备好之后使用编辑软件打开_config.yml

在底部添加 port: 4001

使用cmd进入之前下载好的项目内

然后使用 bundle 来启动 jekyll

```powershell
bundle exec jekyll server
```

这样在启动项目的时候就可以访问 http://127.0.0.1:4001 查看项目状态

```powershell
Configuration file: D:/alfred-alan.github.io/_config.yml
            Source: D:/alfred-alan.github.io
       Destination: D:/alfred-alan.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 11.128 seconds.
 Auto-regeneration: enabled for 'D:/alfred-alan.github.io'
  JekyllAdmin mode: production
    Server address: http://127.0.0.1:4001//
  Server running... press ctrl-c to stop
```

<br>

将写好的md 文件命名为2017-04-18-hello-jekyll.md

文章路径一般存放于 `_posts`

注意 ：有些文件编码可能跟jekyll 不一致 这样就需要更改配置 

一般是在_config.yml 里添加encoding: UTF-8

<br>

如果在本地预览出现not found问题

可能是链接中加入了中文不能被识别

## 解决方法

打开文件

```powershell
Ruby26-x64\lib\ruby\2.6.0\webrick\httpservlet\filehandler.rb
```

在以下两行中加入代码

```ruby
path = req.path_info.dup.force_encoding(Encoding.find("filesystem"))
path.force_encoding("UTF-8") # 加入编码
if trailing_pathsep?(req.path_info)
```

```ruby
while base = path_info.first
    break if base == "/"
    base.force_encoding("UTF-8") #加入編碼
    break unless File.directory?(File.expand_path(res.filename + base))
```

<br>

如果还是解决不了的话 请根据jekyll版本来百度解决 我自己是每个试出来的 误打误撞...

当编写完成后 使用jekyll build打包 在进行提交到指定的库中



## 打包为静态文件

```powershell
jekyll build
```

 生成 每次要提交的事时候也需要生成一下
默认会将网站生成到 ./_site 目录下，生成目录可以通过配置文件 ./_config.yml