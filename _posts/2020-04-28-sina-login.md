---
layout: post
title: '实现微博第三方平台登录'
description: '如何实现使用微博进行登录'
categories: [Python]
tags: [Login]
image: /assets/img/blog/weibo.png

related_posts:
  - _posts/2020-04-26-selenium-login.md
  - _posts/2020-04-26-login-lock.md
---
- Table of Contents
{:toc .large-only}

## 微博开放平台

在微博网站 注册使用app  [https://open.weibo.com/](https://open.weibo.com/)

登录账号选择网站接入

![网站接入](/assets/img/sina_login/website_access.png)

## 创建应用

![创建应用](/assets/img/sina_login/create_app.png)

  应用创建成功后，会立即跳转审核页面，需要填写一些资质，用来进行审核，其实这些审核字段都不用搭理，应用压根就不需要通过审核

![20200311091634_25884](/assets/img/sina_login/20200311091634_25884.png)

之后填写指定的回调路径

![回调信息](/assets/img/sina_login/callback.png)

此时我们有了 `App Key` `App Secret` `回调页` 需要构思编码的流程

## 思路

![思路](/assets/img/sina_login/Ideas.png)


## 跳转微博登录
首先根据 app key 来访问请求API接口 

```js
//新浪微博第三方登录
sina(){
    let client_id=2464168997
    let url="https://api.weibo.com/oauth2/authorize?client_id="+client_id+"&redirect_uri=回调路由"
    window.location.href=url
},
// 这里回调路由就是我们后端的路由
```

## 解析回调参数
在指定回调路由下 接受参数

```python
def wb_back(request):
    # 初始返回一个code
    code=request.GET.get('code',None)
    # 微博接口地址
    access_token_url = "https://api.weibo.com/oauth2/access_token"

    # 定义参数
    re = requests.post(access_token_url, data={
        "client_id": 'app key',
        "client_secret": "secert key",
        "grant_type": "authorization_code",
        "code": code,
        "redirect_uri": "回调路由",
    })

    # 获取用户信息
    params = {
        "access_token": re.json()["access_token"],
        "uid": re.json()["uid"]
    }
    res = requests.get('https://api.weibo.com/2/users/show.json', params)

    # res.json（）就是用户登录的信息
    print(res.json())
```

## 记录用户
上面代码实现了接受用户的参数

下面根据用户参数实现一些判断逻辑

```python
    username=str(res.json()['name'])
    # 判断是否用新浪微博登录过
    user=User.objects.filter(username=username).first()
    sina_id=''
    user_id=''
    if user:
        # 代表曾经登录过
        sina_id=user.username
        user_id=user.id
    else:
        # 首次登陆
        User(username=username,password=res.json()['id']).save()
        user=User.objects.filter(username=username).first()
        sina_id=user.username
        user_id=user.id

    print(sina_id,user_id)
    time.sleep(2)

    # 重定向到前端主页 并传递参数
    return redirect("http://localhost:8080?sina_id="+str(sina_id)+"&uid="+str(user_id))
```


## 记录登录状态
前端解析后端传递的参数

```js
var sina_id = this.$route.query.sina_id;
var user_id = this.$route.query.uid;
if (sina_id){
    // 自动帮用户登录
    localStorage.setItem('username',sina_id)
    localStorage.setItem('uid',user_id)
}
```