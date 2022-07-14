---
layout: post
title: 'Mysql5.7.35配置主从同步'
description: '如何在ubuntu2004下配置Mysql主从同步'
categories: [Mysql]
tags: []
image:  /assets/img/blog/mysql_master_slave.png

related_posts:
  - _posts/2021-11-25-ubuntu-install-mysql.md
  - _posts/2020-7-15-docker-install-mysql.md
---

- Table of Contents
{:toc .large-only}



### 环境

我使用的是两套ubuntu2004 + mysql5.7.35

### 同步数据

master 备份数据

```shell
# 备份所有库
mysqldump -uroot -proot --all-databases --lock-all-tables > ./master_db.sql
# 备份单个库
mysqldump -uroot -proot test_slave --lock-all-tables > ./master_db.sql
```

slave 同步数据

如果是单个数据库 需要先创建相应的数据库然后在进行导入

```shell
# 导入单个数据
mysql -uroot -proot test_slave < ./master_db.sql
# 导入所有数据库
mysql -uroot -proot  < ./master_db.sql
```

同样 也可以使用`source`进行导入

```shell
mysql> source ./Desktop/master_db.sql;
```

### 修改配置文件

我这里主要针对一个库进行同步

如果需要同步所有数据库 需要设置master的`binlog_ignore_db` 和slave的`replicate-do-db`



master配置文件

```shell
server-id               = 1
log_bin                 = /var/log/mysql/mysql-bin.log
#指定存储库
binlog_do_db            = test_slave
# binlog_ignore_db      = include_database_name
```
slave配置文件
```shell
server-id               = 2
report-host             = 主机ip
# 指定接收库
replicate-do-db         = test_slave
#binlog_do_db           = include_database_name
#binlog_ignore_db       = include_database_name
```

### 创建用户

在master上 创建slave用户

```shell
create user 'slave'@'%' identified by 'slave';
```

```shell
grant replication slave, replication client on *.* to 'slave'@'%'; (修改时 加上 with grant option)
```

```shell
flush privileges;
```

查询用户 验证一下

```shell
mysql> select user, host from mysql.user;

+------------------+-----------+
| user             | host      |
+------------------+-----------+
| slave            | %         |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
6 rows in set (0.00 sec)
```

### 设置slave同步

查询master状态

需要记住bin.log的名称 和position的位置

```shell
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000008 |      859 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

在slave上配置同步

```shell
change master to master_host='ip', master_port=3306, master_user='slave', master_password='slave', master_log_file='mysql-bin.000008', master_log_pos=859;
```



查询slave的状态

```shell
show slave status \G;
```

```shell
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000008
          Read_Master_Log_Pos: 859
               Relay_Log_File: clean-pods-1-relay-bin.000008
                Relay_Log_Pos: 859
        Relay_Master_Log_File: mysql-bin.000008
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: test_slave
          Replicate_Ignore_DB: 
```

### 问题

我这里启动了之后 显示状态都为No

尝试跳过一次错误

```shell
stop slave;
```
跳过一次错误
```shell
SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1
```

```shell
start slave
```

再次查看状态 发现成功



### 查看状态

查看slave的状态

```shell
show slave status \G;
```

查看master的状态

```shell
show master status;
```

查看链接的从库

```shell
show slave hosts;
```







