---
layout: post
title: '使用高德API 来进行ip定位 '
description: '如何使用高德api'
categories: [Python]
image: /assets/img/blog/gaode.png
accent_image:  /assets/img/blog/gaode.png
---
- Table of Contents
{:toc .large-only}

## 如何获取ip

在进行客户定位的时候，会遇到一个很经典的问题。

在前端 js 如何根据自身ip 来获取到自身所处的位置？

在网上发现很多教程，大多数都是远古时期的教程 ，只通过外链js 来获取到ip地址。

现在大多数都已经关闭了服务 ，搜狐还在坚持 

## 使用高德API

而这次我们使用一个较为简单且持久的方法，就是使用 高德地图api 来协助定位。

首先要注册，不要嫌麻烦，注册成功就一劳永逸

 高德开发者平台 https://console.amap.com/dev/index

## 创建应用

![创建应用](/assets/img/gaode/create_application.png)
<br/>

![应用姓名](/assets/img/gaode/application_name.png)

## 创建秘钥

![添加key](/assets/img/gaode/Add_key.png)

<br/>

添加之后会生成一个 key 这个要保存好

![记录key](/assets/img/gaode/record_key.png)

## 使用定位功能

之后转到开发指南 https://lbs.amap.com/api/webservice/guide/api/ipconfig

选择 ip 定位 

![ip指南](/assets/img/gaode/ip_guide.png)

## 编写请求

根据上面的说明 ，编写一个简单的axios请求

```javascript
this.axios.get('https://restapi.amap.com/v3/ip?parameters',
               {params:{'key':'应用key'}})
.then(res=>{
           console.log(res.data)
          })
```
## 执行结果

```powershell
{status: "1", info: "OK", infocode: "10000", province: "宁夏回族自治区", city: "银川市", …}
status: "1"
info: "OK"
infocode: "10000"
province: "宁夏回族自治区"
city: "银川市"
adcode: "640100"
rectangle: "105.9809554,38.31814956;106.5117323,38.61089325"
__proto__: Object
```

这就拿到了我的地址 

看是不是很简单 

只是发送一个请求就可以获取到我自己的 ip 加上实际位置

## 使用python请求 

```python
# file: 'test.py'
import requests

res= requests.get('https://restapi.amap.com/v3/ip?parameters',params={'key':'c8165322c2c48892f90ccda1aef617cf'})
print(res.json())
```

只需要三行代码就知道自己所在的位置和坐标拉 

岂不是很简单！