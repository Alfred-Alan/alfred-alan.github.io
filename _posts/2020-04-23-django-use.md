---
layout: post
title: '使用Django实现简单注册功能'
description: '如何配置Django以及实现简单的注册'
categories: [Python]
tags: [Django]
image: /assets/img/blog/django.png
---
- Table of Contents
{:toc .large-only}

## Django了解

Python下有许多款不同的 Web 框架。Django是重量级选手中最有代表性的一位。许多成功的网站和APP都基于Django。

Django是一个开放源代码的Web应用框架，由Python写成

## 下载Django

在已有python环境下 使用 pip install django == 2.0.4 进行下载

我这里是用的2.0.4版本 相对来说还是很稳定

## 创建Django项目

```powershell
django-admin startproject Helloword
```

## 了解Django目录：

```powershell
|-- HelloWorld: 项目的容器。
|   |-- __init__.py:  一个空文件，告诉 Python 该目录是一个 Python 包。
|   |-- settings.py:  该Django项目的设置/配置。
|   |-- urls.py:  该Django项目的URL 声明; 一份由 Django 驱动的网站"目录"。
|   `-- wsgi.py:  一个WSGI兼容的Web服务器的入口，以便运行你的项目
`-- manage.py:  一个实用的命令行工具，可让你以各种方式与该 Django 项目进行交互。
```

## 创建app

cd 到新建的项目的目录下

```powershell
python manage.py startapp myapp
```

会创建一个`myapp`目录 他的目录结构如下：

```powershell
myapp/
    __init__.py   # 说明这是个python包
    admin.py   # 管理后台的文件
    migrations/   # 数据库每次改动都会在这个目录下生成一条记录 
        __init__.py   # 说明这是个python包
    models.py   # models文件，主要编写一些数据库的表结构，字段等
    tests.py   # 测试用的文件
    views.py   # 试图函数的文件，大多数我们是在这个文件进行页面逻辑的编写
```

## 编写第一个view

打开`myapp/views.py` 把下面的代码输入进去：

```python
from django.http import HttpResponse,JsonResponse
from django.views import View

class Show(View):
    def get(self,request):
        return JsonResponse({"code":200,"msg":"HelloWorld"})
```

## 设置settings 并注册app

打开`HelloWorld/settings.py`如下设置

```python 
# INSTALLED_APPS 最后一行添加
'myapp.apps.MyappConfig',

# 将数据库设置为mysql
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': "",
        'HOST':'localhost',
        'PORT':3306,
        "USER":'',
        "PASSWORD":"",
        'OPTIONS': {
             'autocommit': True,
         }
    }
}

# 更改时区
LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

USE_TZ = False
```

## 设置url

打开`HelloWorld/urls.py` 如下设置

```python
from django.contrib import admin
from django.urls import path,include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('myapp/', include('myapp.urls')),
]
```

而后在`myapp`中创建一个`urls.py` 将下面代码输入进去

```python
from django.urls import path,include
from .views import *

urlpatterns = [
    path('show/', Show.as_view(), name='show'),
]
```

## 启动Django

```powershell
python manage.py runserver
```

用你的浏览器访问`http://localhost:8000/myapp/show/`你应该能看见 HelloWorld

## 创建model

常用字段类型:

```
AutoField	一个自动增加的整数类型字段。
BigAutoField	64位整数类型自增字段
BigIntegerField	64位整数字段（看清楚，非自增
BinaryField	二进制数据类型。使用受限，少用
BooleanField	布尔值类型。默认值是None
CharField	字符串类型
DateField	日期类型
DateTimeField	日期时间类型与DateField相比就是多了小时、分和秒的显示
DecimalField	固定精度的十进制小数
EmailField	邮箱类型，默认max_length最大长度254位
FileField	上传文件类型
FilePathField	文件路径类型
FloatField	浮点数类型，参考整数类型
ImageField	图像类型
IntegerField	整数类型
TextField	大量文本内容，在HTML中表现为Textarea标签
TimeField	时间字段同DateField一样的参数，只作用于小时、分和秒。
URLField	一个用于保存URL地址的字符串类型，默认最大长度200。
UUIDField	用于保存通用唯一识别码的字段。使用Python的UUID类
```


打开`myapp/models.py` 写一个简单的模型类

```python
class User(models.Model):
    username = models.CharField(max_length=200, verbose_name="用户名")
    password = models.CharField(max_length=200, verbose_name="密码")
    # 声明表名
    class Meta:
        db_table = "user"
```

写好之后指定app 迁移数据库

```powershell
python manage.py makemigrations myapp
```

执行后你将会看到类似下面的输出

```powershell
Migrations for 'myapp':
  polls/migrations/0001_initial.py
    - Create model User
```

通过运行 `makemigrations` 命令，Django 会检测你对模型文件的修改（在这种情况下，你已经取得了新的），并且把修改的部分储存为一次 *迁移*。

Django 有一个自动执行数据库迁移并同步管理你的数据库结构的命令 -这个命令是 `migrate`

```powershell
python manage.py migrate
```

当数据库无法迁移时 :
将出问题的app下的所有模型，都和数据库中的表保持一致。
将出问题的app下的migrations文件夹删除
将数据库清空 在进行重新迁移

### 例子：实现一个简单的注册

```python
class Register(APIView):
    def get(self, request):
        # 接受参数
        username = request.GET.get('username', None)
        password = request.GET.get('password', None)

        user = User.objects.filter(username=username).first()
        if user:
            return Response({"code": 403, "msg": "该用户名已存在"})
        # 入库
        try:
            user = User(username=username,password=password,)
            user.save()
        except Exception as e:
            return Response({"code": 403, "msg": "注册失败"})
        return Response({"code": 200, "msg": "注册成功"}
```

