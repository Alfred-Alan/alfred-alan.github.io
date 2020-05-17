---
layout: post
title: '反序列化添加及序列化底层'
subtitle: '序列化底层是如何实现'
date: 2020-05-15
categories: 技术
tags: django vue 
image: /assets/img/blog/rest_framework.jpg

---

####  序列化

DRF的核心 就是 前后端分离的核心

**前后端分离开发的核心：**
将模型转换为json 称之为 序列化
将json转换为模型 称之为 反序列化

制作序列化类

```python
from rest_framework import serializers

from myapp.models import *
class userlSer(serializers.ModelSerializer):
    class Meta:
        model=User
        fields = '__all__'
```

序列化底层是根据 sql语句查出的结果集在进行json格式化

我们就来实现一下原理

```python
# 导入原生sql模块
from django.db import connection

# 搜索接口
class Search(APIView):
    def get(self,request):
        # 检索字段
        text = eval(request.GET.get('text',None))
        # 建立游标对象
        cursor=connection.cursor()
        # 执行sql语句
        cursor.execute("select * from goods where name like '%%%s%%'" %text)
        # 查询
        res=dictfetch(cursor)
        # 判断长度
        count=len(res)
        return Response({'msg':text,"data":res,'total':count})

```

在执行查询的部分 我们调用了``dictfetch`` 函数 

因为游标查询出的数据是元祖形式 但是与前端类型不符合

函数内的逻辑就是 将查询出的数据 进行列表嵌套键值对的二次重组

```python
#  格式化结果集
def dictfetch(cursor):
    # 声明描述符 description获取字段名
    desc= cursor.description
    # 重组结果
    return [dict(zip([col[0] for col in desc],row))
        for row in cursor.fetchall()
    ]
```

zip()函数将可迭代对象作为参数，并打包成元组   

之后在使用dict将tuple 转换为键值对

```python
[('name', '2')]
```

#### 反序列化添加

反序列化是将所有参数打包为一个dict   

之后将dict提交给序列化类 通过save就可以直接入库保存了

```python
class UserInsert(APIView):
    def post(self,request):
        # 初始化参数
        name = request.GET.get('name')
        password = request.GET.get('password')
        # 反序列化添加
        data={
            'name':name,
            'password':password
        }
        user = User_Ser(data=data)
        # 验证字段是否错误
        if user.is_valid():
            # 进行入库操作
            user.save()
            
        return Response({'code':200,'msg':'ok'})
```

在反序列化时候一定要注意字段的类型  

要与字段个数一致