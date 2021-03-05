---
layout: post
title: 'django建表及序列化'
description: '如何使用序列化返回数据'
categories: [Python]
tags: [Django]
image: /assets/img/blog/serializes.jpg

related_posts:

---
- Table of Contents
{:toc .large-only}

## 什么是序列化

**将程序中的一个数据结构类型转换为其他格式（字典、JSON、XML等），例如将Django中的模型类对象装换为JSON字符串，这个转换过程我们称为序列化。**

**前后端分离开发的核心：**
将模型转换为json 称之为 序列化
将json转换为模型 称之为 反序列化

## 使用序列化器
首先注册序列化所使用的模块

在``settings`` 中

```python
# file: 'settings.py'
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp.apps.MyappConfig',
    'corsheaders',
    # 注册rest
    'rest_framework',

]
```

## 创建model模型类

```python
# file: 'models.py'
class User(models.Model):
    create_time = models.DateTimeField(blank=True, null=True)
    username = models.CharField(max_length=200)
    password = models.CharField(max_length=200)
    img = models.CharField(max_length=200, blank=True, null=True)
    type = models.IntegerField()
    phone = models.CharField(max_length=200, blank=True, null=True)
    num = models.IntegerField(blank=True, null=True)

    class Meta:
        db_table = 'user'

```

## 迁移数据库

```powershell
python manage.py makemigrations  生成迁移文件

python manage.py migrate  根据迁移文件更改数据库
```

## 创建序列化器

之后在app文件夹内创建一个py文件

```python
# file: 'serializer.py'
from rest_framework import serializers

from myapp.models import *
class userlSer(serializers.ModelSerializer):
    class Meta:
        model=User
        fields = '__all__'
```

意思是将指定的model类生成一个序列化器

这个序列化方法是最简单的

然后在view 中导入

```python
from django.utils.deprecation import MiddlewareMixin
from myapp.serializers import userlSer
```

## 序列化器的使用方式

```python
# file: 'view.py'

# 商品分类接口
class User_show(APIView):
    def get(self,request):
        # 查询数据
        users=User.objects.all()
        # 生成序列化对象
        user_ser=userlSer(users,many=True)
		# 返回数据
        return Response(user_ser.data)
```

以上是可以手写返回方式及数据

## 更简单的方法

```python
# file: 'view.py'

from rest_framework.mixins import  在mixins 中导包
ListModelMixin,       序列化展示
CreateModelMixin,    反序列化创建
DestroyModelMixin,    单个获取
RetrieveModelMixin,    删除
UpdateModelMixin,    修改

from rest_framework.generics import GenericAPIView

class Show(ListModelMixin,GenericAPIView)
    queryset =  指定要查询的集合
    serializer_class =   指定序列化的类
    # 返回序列化信息
    def get(self,request,*args,**kwargs):
        return self.list(request,*args,**kwargs)
```

这样就根本不需要写逻辑了

调用接口返回指定的数据

简单易懂