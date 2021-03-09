---
layout: post
title: 'Flak + Celery 设置定时任务'
description: '如何在flasks上搭建celery定时任务'
categories: [Python]
tags: [Flask,Celery]
image: /assets/img/blog/celery-flask.png

---
- Table of Contents
{:toc .large-only}

## flask结合celery
之前提到了如何在 django环境下配置celery异步定时任务

这次说一下如何在flask中配置celery环境

在我实践代码的时候，发现网上的教程都不相同，有的不能实现，而有的很复杂，没有一个唯一的答案。

所以就说一下比较简单的配置方法。

## 创建celery对象

首先，在``app.py`` 下实例化 celery对象

```python
# file: 'app.py'
from datetime import timedelta
from celery import Celery

# 创建celery对象,设置任务队列使用redis
broker = 'redis://127.0.0.1:6379/0'
backend = 'redis://127.0.0.1:6379/1'
celery = Celery('tasks',broker=broker,backend=backend,
             #包含以下链各个任务文件，去响应的py文件中找任务，对多个任务做分类
             include=['tasks',]
             )

```
## 创建任务
之后创建一个``tasks.py``  写一个简单的任务

```python
# file: 'tasks.py'
from app import celery

# 创建任务
@celery.task
def add(a, b):
    n = a + b
    print(n)
    return n
```
## 设置定时任务
写好之后 在回到``app.py`` 添加定时任务

```python 
# file: 'app.py'
# beat_schedule 定义定时任务的
celery.conf.beat_schedule = {
    'two_nums': {
        # 执行tasks1下的test_celery函数
        'task': 'tasks.add',
        'schedule': timedelta(seconds=30),
        # 传递参数
        'args':(5,6)
    },
}
```

## 调用任务

```python
# file: 'manage.py'
import tasks 
def test():
	res=tasks.add.delay(5,6) # 使用delay 方法
   	print(res)
```

## 开启celery服务

```powershell
celery worker -A tasks -l info -P eventlet
```

这里要说一嘴 可能 celery 普通启动会出错，所以使用eventlet包来协助启动。

启动效果如下

```powershell
(base) E:\Python\myflask_dev>celery worker -A tasks -l info -P eventlet
 -------------- celery@DESKTOP-I4TUILL v4.4.2 (cliffs)
--- ***** -----
-- ******* ---- Windows-10-10.0.17134-SP0 2020-07-12 19:44:50
- *** --- * ---
- ** ---------- [config]
- ** ---------- .> app:         test:0x1796b277780
- ** ---------- .> transport:   redis://127.0.0.1:6379/0
- ** ---------- .> results:     redis://127.0.0.1:6379/1
- *** --- * --- .> concurrency: 12 (eventlet)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** -----
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery


[tasks]
  . tasks.add

[2020-07-12 19:44:50,296: INFO/MainProcess] Connected to redis://127.0.0.1:6379/0
[2020-07-12 19:44:50,305: INFO/MainProcess] mingle: searching for neighbors
[2020-07-12 19:44:51,325: INFO/MainProcess] mingle: all alone
[2020-07-12 19:44:51,336: INFO/MainProcess] pidbox: Connected to redis://127.0.0.1:6379/0.
[2020-07-12 19:44:51,339: INFO/MainProcess] celery@DESKTOP-I4TUILL ready.

```
## 启动定时任务


```powershell
celery -A tasks  beat -l info
```

```powershell
(base) E:\Python\myflask_dev>celery -A tasks  beat -l info
celery beat v4.4.2 (cliffs) is starting.
__    -    ... __   -        _
LocalTime -> 2020-07-12 19:45:11
Configuration ->
    . broker -> redis://127.0.0.1:6379/0
    . loader -> celery.loaders.app.AppLoader
    . scheduler -> celery.beat.PersistentScheduler
    . db -> celerybeat-schedule
    . logfile -> [stderr]@%INFO
    . maxinterval -> 5.00 minutes (300s)
[2020-07-12 19:45:11,149: INFO/MainProcess] beat: Starting...
```

## 结果
当30s 之后 定时任务会启动 结果如下

```powershell
# 定时服务下 提示
[2020-07-12 19:45:41,201: INFO/MainProcess] Scheduler: Sending due task two_nums (tasks.add)


# celery 异步服务下提示
2020-07-12 19:45:41,215: INFO/MainProcess] Received task: tasks.add[98691583-5b8e-45c5-ade8-c0d06fcaeb0d]
[2020-07-12 19:45:41,216: WARNING/MainProcess] 11
[2020-07-12 19:45:41,217: INFO/MainProcess] Task tasks.add[98691583-5b8e-45c5-ade8-c0d06fcaeb0d] succeeded in 0.01600000000325963s: 11
```

同时还可以在redis中查看执行结果

因为结果是存储在数据库1中，输入 ``select 1 `` 切换到 数据库1

之后使用 celery-task-meta- 加上执行id 查询 

```powershell
C:\Users\by wlop>redis-cli
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> get celery-task-meta-98691583-5b8e-45c5-ade8-c0d06fcaeb0d
"{\"status\": \"SUCCESS\", \"result\": 11, \"traceback\": null, \"children\": [], \"date_done\": \"2020-07-12T11:45:41.216875\", \"task_id\": \"98691583-5b8e-45c5-ade8-c0d06fcaeb0d\"}"
```





