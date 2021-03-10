---
layout: post
title: '在Ubuntu上设置定时任务'
description: '如何在Ubuntu上使用定时脚本打包jekyll'
categories: [Ubuntu]
tags: [jekyll]
image:  /assets/img/blog/crontab.png
# accent_image: /assets/img/blog/RabbitMQ-logo.png
# invert_sidebar: true

related_posts:
  - _posts/2021-2-25-jekyll-nginx.md
  - _posts/2020-7-16-ubuntu-add-usergroup.md
---
- Table of Contents
{:toc .large-only}

上文说到使用nginx代理jekyll静态文件，
但是每次都需要登录服务器去打包 迁移 特别麻烦
这回搞一个定时拉取代码 打包的脚本省的每次操作服务器
做一个懒货

## 创建脚本

新建一个make_blog.sh 脚本

```shell
# file: '/root/make_blog.sh'
#!/bin/bash
  
cd /root/alfred-alan.github.io||exit;
git pull origin starter_kit;

build_log=`JEKYLL_ENV=production /usr/local/bin/bundle exec /usr/local/bin/jekyll build`
echo "${build_log}" $$
cp -r /root/alfred-alan.github.io/_site/* /home/nginx/html/;
```
这个地方 ``/usr/local/bin/bundle``
因为在 crontab 中可能找不到指定的命令 我们需要命令的绝对路径

## 测试脚本
写好了 ok 我们来执行一下

```shell
sh make_blog.sh
```

```shell
root@clean-pods-1:~# sh make_blog.sh 
From https://github.com/Alfred-Alan/alfred-alan.github.io
 * branch            starter_kit -> FETCH_HEAD
Already up to date.
Configuration file: /root/alfred-alan.github.io/_config.yml
 Theme Config file: /root/alfred-alan.github.io/#jekyll-theme-hydejack/_config.yml
            Source: /root/alfred-alan.github.io
       Destination: /root/alfred-alan.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
                    done in 8.973 seconds.
 Auto-regeneration: disabled. Use --watch to enable.
```

发现我们打包项目大概需要10-15秒左右 这个要记下来



## 定时任务的操作

Cron** 服务命令：

```shell
# 查看服务状态
sudo  service cron status
# 开启服务
sudo service cron start
# 停止服务
sudo service cron stop
# 重启服务
sudo service cron restart
```

## 编辑定时设置 
```powershell
crontab -e
```

第一次要选择 以哪种编辑器 我们选择 vim 3

输入配置信息

```shell
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
* * * * * /root/make_blog.sh >/root/make_log.txt 2>&1
* * * * * sleep 15; /root/make_blog.sh >/root/make_log.txt 2>&1
* * * * * sleep 30; /root/make_blog.sh >/root/make_log.txt 2>&1
* * * * * sleep 45; /root/make_blog.sh >/root/make_log.txt 2>&1
```

sleep 代表延迟多少秒运行

上面例子就是每15秒进行一次 然后把结果放到 /root/make_log.txt里面 方便我们查看

## 重启cron

```powershell
sudo service cron reload
```

## 修改rsyslog
现在我们要看 crontab执行的情况

首先修改rsyslog

```powershell
vim /etc/rsyslog.d/50-default.conf
```
```powershell
#  Default rules for rsyslog.
# 
#                       For more information see rsyslog.conf(5) and /etc/rsyslog.conf

#
# First some standard log files.  Log by facility.
#
auth,authpriv.*                 /var/log/auth.log
*.*;auth,authpriv.none          -/var/log/syslog
#cron.*                          /var/log/cron.log
#daemon.*                       -/var/log/daemon.log
kern.*                          -/var/log/kern.log
#lpr.*                          -/var/log/lpr.log
mail.*                          -/var/log/mail.log
#user.*                         -/var/log/user.log

```

搜索cron 把如下行之前的注释"#"去掉
```powershell
cron.*              /var/log/cron.log 
```

## 重启rsyslog
```powershell
sudo  service rsyslog  restart
```

## 查看执行情况
现在看看定时任务的日志
```powershell
tail -f /var/log/cron.log
```

可以看到脚本的执行情况
```powershell
root@clean-pods-1:~# tail -f /var/log/cron.log
Mar 10 08:13:02 clean-pods-1 CRON[137804]: (root) CMD (sleep 30; /root/make_blog.sh >/root/make_log.txt 2>&1 )
Mar 10 08:13:02 clean-pods-1 CRON[137805]: (root) CMD (sleep 15; /root/make_blog.sh >/root/make_log.txt 2>&1)
Mar 10 08:14:01 clean-pods-1 CRON[138355]: (root) CMD (sleep 15; /root/make_blog.sh >/root/make_log.txt 2>&1)
Mar 10 08:14:01 clean-pods-1 CRON[138354]: (root) CMD (sleep 30; /root/make_blog.sh >/root/make_log.txt 2>&1 )
Mar 10 08:14:01 clean-pods-1 CRON[138356]: (root) CMD (sleep 45; /root/make_blog.sh >/root/make_log.txt 2>&1)
Mar 10 08:14:01 clean-pods-1 CRON[138357]: (root) CMD (/root/make_blog.sh >/root/make_log.txt 2>&1)
Mar 10 08:15:01 clean-pods-1 CRON[138907]: (root) CMD (sleep 15; /root/make_blog.sh >/root/make_log.txt 2>&1)
Mar 10 08:15:01 clean-pods-1 CRON[138906]: (root) CMD (/root/make_blog.sh >/root/make_log.txt 2>&1)
Mar 10 08:15:01 clean-pods-1 CRON[138908]: (root) CMD (sleep 45; /root/make_blog.sh >/root/make_log.txt 2>&1)
Mar 10 08:15:01 clean-pods-1 CRON[138909]: (root) CMD (sleep 30; /root/make_blog.sh >/root/make_log.txt 2>&1 )
```

## 查看执行结果
我们要查看脚本是否正确 可以查看log
```powershell
cat /root/make_log.txt
```

