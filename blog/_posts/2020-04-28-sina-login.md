---
layout: post
title: '实现微博第三方平台登录'
subtitle: '如何实现使用微博进行登录'
date: 2020-04-28
categories: 技术
tags: sina login
image: /assets/img/blog/weibo.jpg
---

## 实现微博第三方登录

在微博网站 注册使用app[https://open.weibo.com/](https://open.weibo.com/)

登录账号选择网站接入

![网站接入](/assets/img/day8/网站接入.png)

并创建应用

![创建应用](/assets/img/day8/创建应用.png)

  应用创建成功后，会立即跳转审核页面，需要填写一些资质，用来进行审核，其实这些审核字段都不用搭理，应用压根就不需要通过审核

![20200311091634_25884](/assets/img/day8/20200311091634_25884.png)

之后填写指定的回调路径

![回调信息](/assets/img/day8/回调信息.png)

此时我们有了 `App Key` `App Secret` `回调页` 需要构思编码的流程

![思路](/assets/img/day8/思路.png)

首先根据 app key 来访问请求API接口 

```js
//新浪微博第三方登录
sina(){
    let client_id=2464168997
    let url="https://api.weibo.com/oauth2/authorize?client_id="+client_id+"&redirect_uri=回调路由"
    window.location.href=url
},
```

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
    res=requests.get('https://api.weibo.com/2/users/show.json',params={"access_token":re.json()["access_token"],"uid":re.json()["uid"]})
    # res.json（）就是用户登录的信息
    print(res.json())
```

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
    # 重定向到主页 并传递参数
    return redirect("http://localhost:8080?sina_id="+str(sina_id)+"&uid="+str(user_id))
```

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