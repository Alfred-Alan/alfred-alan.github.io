---
layout: post
title: '使用django+gitee实现第三方登录'
description: '如何配合gitee登录接口 实现第三方登录'
categories: [Python]
tags: [Login]
image: /assets/img/blog/gitee.png

related_posts:
  - _posts/2020-04-28-sina-login.md
  - _posts/2020-04-26-login-lock.md
---
- Table of Contents
{:toc .large-only}

## Gitee

码云 gitee平台 是国内版的github 

有时github因为跨国访问的原因，速度很慢

为了满足国内编码人员的需求，开发出了gitee平台，同理与 github 使用方式是一样的。

## 实现gitee的第三方登录

<br/>

首先点击个人设置

找到第三方应用

![第三方应用](/assets/img/gitee_login/another.png)

<br/>

点击创建应用

![创建应用](/assets/img/gitee_login/create_app.png)

<br/>

```Client ID```和 ```Client Secret``` 要记好 后面要用到![应用页面](/assets/img/gitee_login/app_page.png)

<br/>

## 登录流程
登录的基本流程为

![流程](/assets/img/gitee_login/process.png)

<br/>

官方文档：https://gitee.com/api/v5/oauth_doc#/

跟着官方文档的流程走

## 设置跳转链接

```html
<a href="https://gitee.com/oauth/authorize?client_id=应用id&redirect_uri=回调url&response_type=code">
    <img src="图片链接" alt="加载失败" width="80px">
</a>
```

点击图片跳转到官方登录页面

输入密码回调到redirect_url 会传输一个code

```python
code=request.GET.get('code',None)
```
## 获取token
我们根据这个code 来访问gitee 请求到```access_token```

```python
# file: 'views.py'

data={
    'grant_type':'authorization_code',
    'code':code,
    'client_id':'应用id',
    'redirect_uri':'django回调url',
    'client_secret':'应用secret',
}
# 获取登录后的access_token
res=requests.post(url='https://gitee.com/oauth/token',data=data)
token=res.json()['access_token']
```

在拿到 token的时候 需要点击上面的文档链接

找到API 文档内的获取用户资料 要先点击申请授权

![授权获取](/assets/img/gitee_login/authorize.png)

<br/>

## 获取用户数据

```python
# file: 'views.py'

 # 获取用户数据
 ret=requests.get('https://gitee.com/api/v5/user?access_token='+token)
 name = ret.json()['name']
```

这样就获取到了用户的参数 

可以进行一些入库验证操作

```python
# file: 'views.py'

# 查询用户
user=User.objects.filter(username=name).first()
# 如果曾经登录过
if user:
    gitee_id=user.username
    user_id= user.id
    user_type = user.type
else:
    # 新创建用户
    User(username=ret.json()['name'],password=mytools.md5_str(ret.json()['name']),img=ret.json()['avatar_url']).save()
    user = User.objects.filter(username=name).first()
    gitee_id = user.username
    user_id = user.id
    user_type = user.type

return redirect("http://localhost:8080?another_id=" + str(gitee_id) + "&uid=" + str(user_id)+"&type="+str(user_type))
```

## 小结

一共分为三个流程 

通过url 跳转到第三方登录界面

回调函数中 获取code 根据code请求到 access_token

最后在使用 access_token 查询到 用户资料从而进行入库操作

只要理清逻辑 还是很简单的

