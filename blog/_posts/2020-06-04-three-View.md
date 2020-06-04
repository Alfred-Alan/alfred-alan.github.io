---
layout: post
title: 'View APIView 与 GenericAPIView'
subtitle: '详解三种View的区别'
date: 2020-06-04
categories: 技术
tags: django 
image: /assets/img/blog/restframework.jpg


---

# View APIView 与 GenericAPIView

## View

基于主类的基本视图。 所有其他基于类的视图都从该基类继承。 严格来说，它不是通用视图，因此也可以从django.views导入。

**方法流程图**

* --dispatch()

* --http_method_not_allowed()

* --options()

<br/>

**views.py**：

```python
from django.http import HttpResponse
from django.views import View

class MyView(View):

    def get(self, request, *args, **kwargs):
        return HttpResponse('Hello, World!')
```

**urls.py**：

```python
from django.urls import path

from myapp.views import MyView

urlpatterns = [
    path('myview/', MyView.as_view(), name='MyView'),
]
```

该视图可以接受请求的列表

```python
['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']
```

**Methods**

**dispatch**（*request*，** args*，*** kwargs*)

`view`视图接受`request` 参数的函数 并返回HTTP响应。

实现检查HTTP方法，并将请求发送给与HTTP方法匹配的方法。一个`GET`将被委托给`get()`，一个`POST`给`post()`等。

<br/>

**http_method_not_allowed**（*request*，** args*，*** kwargs*）

如果使用不支持的HTTP方法调用该视图，则改为调用此方法。

以纯文本形式返回允许接受的方法列表。

<br/>

**options**（*request*，** args*，*** kwargs*）

处理对OPTIONS HTTP动词的请求的响应。返回带有`Allow`的响应，里面包含视图的允许HTTP方法的列表。

<br/>

## APIView

APIView是django rest framework框架中的基类 APIView继承了django中的View 在View的功能上进行封装

配合 rest framework 的序列化器 serializer 来使用  将数据序列化为json格式返回 

1、传入到视图方法中的是REST framework的Request对象，而不是Django的HttpRequeset对象；

2、视图方法可以返回REST framework的Response对象，视图会为响应数据设置符合前端要求的格式；

3、任何APIException异常都会被捕获到，并且处理成合适的响应信息；

```python
# APIView_test.py
class SnippetList(APIView):

    def get(self, request):
        users = User.objects.all()
        serializer = UserSerializer(users, many=True)
        return Response(serializer.data)
```

## GenericAPIView

GenericAPIView 继承自 APIView，主要增加了操作序列化和数据库查询的方法，作用是为下面 Mixin 扩展类的执行提供方法支持。通常在使用时，可搭配一个或多个 Mixin 扩展类。

<br/>

指明使用的数据查询集

+ queryset

指明视图使用的序列化器

+ serializer_class

<br/>

GenericAPIView 的五个拓展类提供了五个方法分别进行增删改查的不同操作。

搭配 GenericAPIView 使用

- ListModelMixin 提供 list 方法快速实现列表视图
- CreateModelMixin 提供 create 方法快速实现创建资源的视图
- RetrieveModelMixin 提供 retrieve 方法，可以快速实现返回一个存在的数据对象(传入pk)
- UpdateModelMixin 提供 update 方法，可以快速实现更新一个存在的数据对象。实现局部更新
- DestroyModelMixin 提供 destroy 方法，快速实现删除一个存在的数据对象

​	<br/>

使用GenericAPIView可以使我们的代码更简洁

例如上述代码，我们可以简化为

```python
# GenericAPIView_test.py
from rest_framework.mixins import ListModelMixin,CreateModelMixin
from rest_framework.generics import GenericAPIView

class SnippetList(ListModelMixin,CreateModelMixin，GenericAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer

    def get(self, request, *args, **kwargs):
        # 我们不用在get post两个方法中都去ORM调用以及序列化调用
        return self.list(*args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(*args, **kwargs)
```

通过继承GenericAPIView，对类的queryset与serializer_class属性赋值，然后get方法可以调用父类的ListModelMixin方法的list()方法，而post方法可以调用父类CreateModelMixin的方法。

APIViwe 和 GenericAPIView 都是为了使用者更好的编写 符合restful 风格接口

<br/>

### 遇到的坑

在之前的编码中 使用APIView作为基类，发现put 和 get 方法都能使用 request.GET或 request.POST 来获取参数

而当我使用View作为基类的时候 put 方法使用 request.GET或 request.POST 怎么都获取不到

只能通过使用 QueryDict来获取参数 相对来说还是麻烦了不少

```python
from django.http import QueryDict

def put(self,request):
	put = QueryDict(request.body)
```

而APIView 直接把 request 传过来的参数给封装好了  在别的请求方式内都能获取到参数

之前一直以为 APIView 只是配合序列化器来返回序列化数据的方式，才发现对request的参数也会进行封装

使用正常的View来通过 put 方式获取参数难免会遇到一些坑 

