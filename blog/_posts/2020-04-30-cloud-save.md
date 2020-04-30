---
layout: post
title: 'vue+七牛云实现云端存储'
subtitle: '为了节约空间 使用云端存储'
date: 2020-04-30
categories: 技术
tags: 七牛云 vue
image: /assets/img/blog/qiniu.jpg
---

## 七牛云 云端存储

当今有很多云服务 比如什么百度云 阿里云 可以自己的文件存储到云端服务器进行保存

好处是 防止占用空间 防止数据丢失 一劳永逸

而我们结合云端存储 将数据存储到云端服务器中 而就防止我们自己的服务器的空间占用大

当用户使用数据时 服务器就像中间桥一样进行链接  用户-服务器-云端数据

而我们思路就是在用户上传文件的时候就上传到云端，而本地数据库只是更改一下文件名

这样在访问的时候根据域名+文件名就可以展示出文件

#### 使用七牛云

#### ![思路图](/assets/img/day10/思路图.png)

先到七牛云注册账号[七牛云](https://www.qiniu.com/)

之后选择 产品-对象存储

选择使用对象存储并创建空间![创建空间](/assets/img/day10/创建空间.png)

之后点击秘钥

![秘钥](/assets/img/day10/秘钥.png)

python根据秘钥来请求token

![查看秘钥](/assets/img/day10/查看秘钥.png)

python中有一个qiniu的模块 

可以使用该模块 直接请求到token


```python
# 七牛token
from qiniu import  Auth
# qiniu 对象会自动请求token

def get(self,request):
    # 声明认证对象
    q = Auth(access_key='4CmF2Dy-Xi7PQwHm7Ovpk7YlOiw2pqzUqyf7ybhA',
             secret_key='y3tlCWWdvmgsm2a9j6Plu_WWM4b0jFRTD-f3bHZ7')
    # 获取token
    token=q.upload_token('md1907rgzn')
    return Response({'token':token})
```

然后前端vue 接受到对应的token 并存入全局变量

```js
get_token(){
    this.axios.get("http://127.0.0.1:8000/myapp/qiniu/").then(result => {
        console.log(result.data.token)
        this.token = result.data.token
    });
},
```

对表单设置提交按钮的函数

```js
// 存入七牛
upload_qiniu:function(){
    //获取文件
    var img=document.getElementById("img")
    //声明表单参数
    let param = new FormData();
    param.append("img",img.files[0])
    param.append('token',this.token);
    //自定义axios 不允许传输cookie
    const axios_qiniu = this.axios.create({withCredentials:false});
    //发送请求
    axios_qiniu({
        method:'POST',
        url:'http://up-z1.qiniup.com/',  // 七牛域名
        data:param,
        timeout:30000  //超时设置
    }).then(result =>{
        if(result.status==200){
            // 拼接七牛空间的域名 进行展示
            this.src = "http://q9ksotoly.bkt.clouddn.com/"+result.data.key;
            // this.videosrc = config['baseurl']+result.data.key;
        }
    });
},
```

