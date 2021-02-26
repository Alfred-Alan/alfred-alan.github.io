---
layout: post
title: 'redis实现登录5次锁和session存储问题'
description: '如何使用redis来进行登录5次判断,session的存储机制'
categories: [Python]
tags: [Redis,Django,Login]
image: /assets/img/blog/redis.png
related_posts:
  - _posts/2020-04-26-selenium-login.md
  - _posts/2020-04-28-sina-login.md
---

- Table of Contents
{:toc .large-only}

在一些常见的网站 都会有登录次数

我们使用redis来进行登录次数验证 从而实现对账户上锁

## 思路：

对每个注册后的用户在redis记录为0

在每次登录错误时+1 登录成功再次为0

当登录错误为5时 在`redis`添加一条对应的数据`username:time` 指改昵称锁住多少time

```python
class Login(APIView):
    def get(self, request):
        username = request.GET.get('username', None)
        password = request.GET.get('password', None)

        # 链接redis
        r = redis.Redis('localhost', 6379)
        # 从redis查询判断是否有锁
        if r.exists("%s-lock" % username):
            return Response({"code": 403, "msg": "登录次数过多 已锁住1分钟"})

        user = User.objects.filter(username=username, password=make_password(password)).first()
        # 判断user
        usercount=r.get(username)
        if user:
            # 当登录成功时 次数清0
            r.set(username, 0)
            return Response({"code": 200, "msg": "登录成功"})
        else:
            # 当该用户次数查询不到
            if usercount == None:
                return Response({"code": 403, "msg": "用户名或者密码错误"})
            # 登录失败 次数+1
            count = int(usercount.decode('utf-8'))
            if count == 5:
                r.setex("%s-lock" % username, 60, 1)  # 设置登录锁
                return Response({"code": 403, "msg": "登录次数过多 已锁住1分钟"})
            count += 1
            r.set(username, count)
            return Response({"code": 403, "msg": "用户名或者密码错误"})
```

## django 与vue 前后端分离取不到 session问题 

django在存入session后 取到session为none

是因为session基于cookie来实现 而前端在axios请求的时候没有加上cookie

需要在main.js中加上

```js
Axios.defaults.withCredentials = true;
```

这样使axios在每次请求都加上cookie

django后端才能根据cookie 从表中拿出session

