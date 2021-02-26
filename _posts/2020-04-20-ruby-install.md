---
layout: post
title: '如何安装ruby'
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
* Rubygems
* jekyll

<br>

ruby官网下载安装：<https://rubyinstaller.org/downloads/>

ruby+devkit 2.6.6-1(x64)
{:.note title="注意安装"}
<br>
安装完成后配置环境变量

在命令提示符中，得到ruby版本号，如下图，即安装成功

```powershell
C:\Users\haoyang>ruby -v
ruby 2.6.6p146 (2020-03-31 revision 67876) [x64-mingw32]
```

## 安装RubyGems

官网下载 <http://rubygems.org/pages/download>

rubygems-3.2.4.zip
{:.note title="注意安装"}

<br>
cd到RubyGems目录

```powershell
E:\xunleirubygems-3.2.4>
```

执行安装

```powershell
E:\xunlei\rubygems-3.2.4>ruby setup.rb
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
我这边是安装了一个jekyll

```powershell
C:\Windows\system32>gem install jekyll
```
安装bundle
```powershell
C:\Windows\system32>gem install bundler

C:\Windows\system32>bundle install
```
<br>
然后下载一个IDE
我这边使用的是[RubyMine](https://www.jetbrains.com/ruby/download/)

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
