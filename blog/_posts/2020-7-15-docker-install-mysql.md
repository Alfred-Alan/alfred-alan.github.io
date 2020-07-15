---
layout: post
title: 'Docker中下载mysql及使用'
subtitle: '如何在Docker中下载mysql并且使用'
date: 2020-07-15
categories: 技术
tags: ubuntu docker mysql  
image: /assets/img/blog/docker.png


---

首先在docker中下载mysql

```powershell
docker pull mysql 
```

然后查看镜像

```powershell
docker images -a 
```

接下来开启服务

```powershell
 docker run -p 3306:3306 --name L_mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql
```

--name 是 自定的服务名字

MYSQL_ROOT_PASSWORD 就是root 用户的密码 

开启成功查看在运行的容器

```powershell
docker ps -a
```

如何进入到docker开启的mysql服务

```powershell
 docker exec -it 容器名 bash
```

成功之后就可以看到以下界面

```powershell
(base) bywlop@bywlop-F117-B:~/桌面$ docker exec -it L_mysql bash

root@69cfc14e37ee:/# mysql -uroot -p     
Enter password: 


Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> 
```





