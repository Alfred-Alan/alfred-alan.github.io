---
layout: post
title: 'Ubuntu 安装 Mysql5.7.35'
description: '如何在ubuntu2004下 手动安装mysql-5.7.35'
categories: [Ubuntu, Mysql]
tags: []
image:  /assets/img/blog/install-mysql-ubuntu.png

related_posts:
  - _posts/2020-7-15-docker-install-mysql.md
---

- Table of Contents
{:toc .large-only}

### 下载mysql deb包

下载官方5.7 tar.gz 包 选择5.7.35版本

https://downloads.mysql.com/archives/community/



![image-20211216174029752](/assets/mysql.assets/image-20211216174029752.png)



这里我默认下载到 Downloads里面

```bash
cd ~/Downloads
```

创建文件夹

```shell
mkdir mysql5.7.35
```

解压到指定文件夹

```bash
tar -xvf mysql-server_5.7.35-1ubuntu18.04_amd64.deb-bundle.tar -C ./mysql5.7.35 
```

```bash
libmysqlclient20_5.7.35-1ubuntu18.04_amd64.deb
libmysqlclient-dev_5.7.35-1ubuntu18.04_amd64.deb
libmysqld-dev_5.7.35-1ubuntu18.04_amd64.deb
mysql-client_5.7.35-1ubuntu18.04_amd64.deb
mysql-common_5.7.35-1ubuntu18.04_amd64.deb
mysql-community-client_5.7.35-1ubuntu18.04_amd64.deb
mysql-community-server_5.7.35-1ubuntu18.04_amd64.deb
mysql-community-source_5.7.35-1ubuntu18.04_amd64.deb
mysql-community-test_5.7.35-1ubuntu18.04_amd64.deb
mysql-server_5.7.35-1ubuntu18.04_amd64.deb
mysql-testsuite_5.7.35-1ubuntu18.04_amd64.deb
```

删除测试文件

```bash
rm *test*
```



### 开始安装

#### 1. 安装 `mysql-common`

```bash
sudo dpkg -i mysql-common_5.7.35-1ubuntu18.04_amd64.deb 
```



#### 2. 安装`mysql-community-client`

会提示缺少 `libtinfo5`依赖包
```bash
sudo apt install libtinfo5
```

再安装

```bash
sudo dpkg -i mysql-community-client_5.7.35-1ubuntu18.04_amd64.deb
```




#### 3. 安装`mysql-client`

```bash
sudo dpkg -i mysql-client_5.7.35-1ubuntu18.04_amd64.deb
```




#### 4. 安装`mysql-community-server`

提示缺少`libmecab2` 依赖包
```bash
sudo apt install libmecab2
```
安装
```bash
sudo dpkg -i mysql-community-server_5.7.35-1ubuntu18.04_amd64.deb
```

安装完成后会提示设置root密码




#### 5. 查看状态


安装服务完成了 查看一下运行状态

```bash
service mysql status 
```

```bash
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset:>
     Active: active (running) since Tue 2021-12-28 02:29:08 CST; 8h ago
    Process: 1189 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=e>
    Process: 1285 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/my>
   Main PID: 1289 (mysqld)
      Tasks: 29 (limit: 18880)
     Memory: 205.1M
     CGroup: /system.slice/mysql.service
             └─1289 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mys>

12月 28 02:29:08 F117-B systemd[1]: Starting MySQL Community Server...
12月 28 02:29:08 F117-B systemd[1]: Started MySQL Community Server.
lines 1-13/13 (END)
```



### 配置mysql

#### 基础配置

由于手动安装的mysql没有基础配置文件 我们需要设置一下mysql的基础配置

```shell
# file: '/etc/mysql/mysql.conf.d/mysqld.cnf'
# Copyright (c) 2014, 2021, Oracle and/or its affiliates.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License, version 2.0,
# as published by the Free Software Foundation.
#
# This program is also distributed with certain software (including
# but not limited to OpenSSL) that is licensed under separate terms,
# as designated in a particular file or component or in included license
# documentation.  The authors of MySQL hereby grant you an additional
# permission to link the program and your derivative works with the
# separately licensed software that they have included with MySQL.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License, version 2.0, for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

[mysqld_safe]
socket		= /var/run/mysqld/mysqld.sock # 以默认配置文件路径为准
nice		= 0

[mysqld]
#
# * Basic Settings
#
user		= mysql
pid-file	= /var/run/mysqld/mysqld.pid # 以默认配置文件路径为准
socket		= /var/run/mysqld/mysqld.sock # 以默认配置文件路径为准
port		= 3306
basedir		= /usr
datadir		= /var/lib/mysql
tmpdir		= /tmp
lc-messages-dir	= /usr/share/mysql

# Disabling symbolic-links is recommended to prevent assorted security risks
skip_symbolic_links=yes

#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
#bind-address		= 127.0.0.1
#
# * Fine Tuning
#
key_buffer_size		= 16M
max_allowed_packet	= 16M
thread_stack		= 192K
thread_cache_size       = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam-recover-options  = BACKUP
#max_connections        = 100
#table_cache            = 64
#thread_concurrency     = 10
#
# * Query Cache Configuration
#
query_cache_limit		    = 1M
query_cache_size        = 16M
#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file        = /var/log/mysql/mysql.log
#general_log             = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log # 以默认配置文件路径为准
#
# Here you can see queries with especially long duration
#log_slow_queries	= /var/log/mysql/mysql-slow.log
#long_query_time = 2
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
server-id			      = 1
log_bin				      = /var/log/mysql/mysql-bin.log
expire_logs_days	  = 10
max_binlog_size     = 100M
#binlog_do_db		= include_database_name
#binlog_ignore_db	= include_database_name
#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!
#
# * Security Features
#
# Read the manual, too, if you want chroot!
# chroot = /var/lib/mysql/
#
# For generating SSL certificates I recommend the OpenSSL GUI "tinyca".
#
# ssl-ca=/etc/mysql/cacert.pem
# ssl-cert=/etc/mysql/server-cert.pem
# ssl-key=/etc/mysql/server-key.pem
```

#### 用户配置

启动配置引导项

```bash
sudo mysql_secure_installation
```

```bash
Securing the MySQL server deployment.

Enter password for user root: 

# 设置 VALIDATE PASSWORD 插件
VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No: y

# 密码验证级别
There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 0
Using existing password for root.

# 修改root密码
Estimated strength of the password: 25 
Change the password for root ? ((Press y|Y for Yes, any other key for No) : n

# 删除其他用户
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.
# 仅允许root本地登录
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.
# 删除测试数据库
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.
 - Removing privileges on test database...
Success.
# 重新加载权限
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
```



##### 修改密码长度

密码策略最低要求8位  为了方便可以改自己适合的长度 不需要可以略过

查看密码策略.

```bash
SHOW VARIABLES LIKE "%password%";
```

修改密码长度

```bash
SET GLOBAL validate_password_length=5;
```


##### 创建用户
上面我设置了root用户不允许外部访问 重新创建一个用户

创建用户并赋予权限

```bash
grant all privileges on * . * to 'username'@'%' identified by "password" with grant option;
```

刷新权限


```bash
FLUSH PRIVILEGES;
```

