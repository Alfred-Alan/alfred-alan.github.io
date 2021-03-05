---
layout: post
title: 'django+celery执行异步操作'
description: '如何celery异步框架'
categories: [Python]
tags: [celery]
image: /assets/img/blog/celery.jpg

related_posts:
  - _posts/2020-04-23-django-use.md
---

- Table of Contents
{:toc .large-only}

## 曾经使用celery踩坑

早在去年就做过使用celery结合django异步发邮件。

在当时 我使用的是 celery 3.x 加上django 2.0.4，python 3.7。

遇到了著名的 ansy 关键字错误，还有celery-with-redis不兼容错误。

给我的印象 太恶心。

## 重蹈覆辙

听说 celery 4.x 修复了一些BUG 

今天重新捡起来

## 安装模块

```powershell
pip3 install celery==4.4.2
pip3 install eventlet==0.25.2
pip3 install Django==2.0.4
```

在 4.x 以上 就不需要 celery-with-redis 了

可以配合 eventlet 来使用

## 配置settings.py文件:

```python
# file: 'settings.py'
CELERY_BROKER_URL = 'redis://localhost:6379/'

CELERY_RESULT_BACKEND = 'redis://localhost:6379/'

CELERY_RESULT_SERIALIZER = 'json'
```
## 创建celery app
在```settings```同级目录创建一个 celery.py

```python
# file: 'celery.py'

# -*- coding: utf-8 -*-
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# 设置环境变量
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mydjango.settings')

# 注册Celery的APP
app = Celery('mydjango')
# 绑定配置文件
app.config_from_object('django.conf:settings', namespace='CELERY')

# 自动发现各个app下的tasks.py文件
app.autodiscover_tasks()
```

## 注册celery
将同级目录的```__init__``` 添加

```python
# file: '__init__.py'
from .celery import app as celery_app
__all__ = ['celery_app']
```

## 创建任务文件
在app中创建```tasks.py```

```python
# file: 'tasks.py'
from mydjango.celery import app

@app.task
def write_log():
	print("123")
	return '成功'
```
## 在视图中调用

```python
# file: 'views.py'
from myapp import tasks
def test():
	res=tasks.Send_Mail.delay()
	return JsonResponse({'code':200, 'task_id': res.task_id}) 
```

## 启动服务
先启动django服务

之后在manage.py 同级目录下 启动celery服务

```powershell
celery worker -A mydjango -l info -P eventlet
```

<br/>

## 访问接口 

![task_id](/assets/img/celery/task_id.png)

<br/>

这个task_id  是存在redis中的名称

可以通过 celery-task-meta- + task_id 来查询

```powershell
C:\Users\by wlop>redis-cli
127.0.0.1:6379> keys *
  1) "celery-task-meta-3c82ba29-1cbf-4dc6-9db7-e4a44277efd9"
  2) "celery-task-meta-e06532e5-c99f-4ee3-9544-3c4100164030"
```

redis中保存的是异步函数执行的信息

<br/>

### 踩坑实践

对函数传参是在 delay里面

```python
res=tasks.Send_Mail.delay(参数)
```

当使用POST 请求 来执行异步函数的时候 会报出

```powershell
TypeError: wrap_socket() got an unexpected keyword argument '_context'
```

为了解决这个情况

```powershell
改变服务器启动方法不要用eventlet，加个参数
celery worker -A celery_name --loglevel=info --pool=solo
```
## 添加定时任务
还可以添加 定时任务

在```settings``` 中添加

```python
# file: 'settings.py'

# 导入celery 定时模块
from celery.schedules import crontab
from datetime import timedelta

CELERY_BEAT_SCHEDULE = {
    # 定义定时任务
    'celery_work': {
        		# 指定定时函数名
        'task': "myapp.tasks.write_log",
        		# 每三十秒执行一次
        'schedule': timedelta(second=30),
        'args': (参数),
    },
}
```

## 启动定时任务
如果需要启动定时任务  在celery服务启动的情况下新建一个服务窗口

```powershell
celery -A mydjango beat -l info
```

之后可以在 celery服务窗口看到每30s 执行一次

<br/><br/>

## 小结

记录了之前的错误，结合现在 4.x 以上配合 eventlet 完美实现异步功能,

同时记录了还有定期执行功能