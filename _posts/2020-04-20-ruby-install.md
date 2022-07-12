---
layout: post
title: '如何安装ruby配置jekyll'
categories: [Ruby]
image: /assets/img/blog/Ruby.png
description: '如何安装ruby并配置环境'
related_posts:
  - _posts/2020-04-20-github+jekyll for blog.md
---

- Table of Contents
{:toc .large-only}

## 安装Ruby

我们需要下载安装以下几个工具

* Ruby
* jekyll

<br>

ruby官网下载安装：<https://rubyinstaller.org/downloads/>

[Ruby+Devkit 2.7.6-1 (x64)](https://github.com/oneclick/rubyinstaller2/releases/download/RubyInstaller-2.7.6-1/rubyinstaller-devkit-2.7.6-1-x64.exe)

{:.note title="我的安装版本"}
<br>

![install_path](/assets/img/ruby/install_path.png)

![select_components](/assets/img/ruby/select_components.png)

![install_msys2](/assets/img/ruby/install_msys2.png)

![select_3](/assets/img/ruby/select_3.png)

输入3之后等待直接结束，可能会卡住很长时间

![install_end](/assets/img/ruby/install_end.png)


安装完成后在命令提示符中，得到ruby版本号，如下图，即安装成功

```powershell
C:\Users\admin>ruby -v
ruby 2.7.6p219 (2022-04-12 revision c9c2245c0a) [x64-mingw32]

C:\Users\admin>gem -v
3.1.6

C:\Users\admin>
```



## 使用RubyGems

在使用gem安装的时候  我遇到了一些问题  原本的源特别慢   我们就需要更改gem下载源

```powershell
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

安装jekyll bundle

```powershell
gem install jekyll bundler
```
<br>
然后下载一个IDE
我使用的是[RubyMine](https://www.jetbrains.com/ruby/download/)

安装完成后打开软件创建一个项目
![rubymine](/assets/img/ruby/rubymine.png)
<br>

新建一个 *.rb的文件
编写输出Hello World

```ruby
puts "Hello World!";
```
<br>
![rubymine](/assets/img/ruby/ruby_print.png)