---
layout: post
title: '使用高德API 来进行ip定位 '
subtitle: '如何使用高德api'
date: 2020-07-13
categories: 技术
tags: js python 
image: /assets/img/blog/gaode.jpg

---

在进行客户定位的时候，会遇到一个很经典的问题。

在前端 js 如何根据自身ip 来获取到自身所处的位置？

在网上发现很多教程，大多数都是远古时期的教程 ，只通过外链js 来获取到ip地址。

现在大多数都已经关闭了服务 ，搜狐还在坚持 

而这次我们使用一个较为简单且持久的方法，就是使用 高德地图api 来协助定位。

首先要注册，不要嫌麻烦，注册成功就一劳永逸

 高德开发者平台 https://console.amap.com/dev/index

然后创建一个应用

![创建应用](/assets/img/gaode/创建应用.png)
<br/>

![应用姓名](/assets/img/gaode/应用姓名.png)

<br/>

为该应用创建一个key

![添加key](/assets/img/gaode/添加key.png)

<br/>

添加之后会生成一个 key 这个要保存好

![记录key](/assets/img/gaode/记录key.png)

<br/>

之后转到开发指南 https://lbs.amap.com/api/webservice/guide/api/ipconfig

选择 ip 定位 

![ip指南](/assets/img/gaode/ip指南.png)

<br/>

根据上面的说明 ，编写一个简单的axios请求

```javascript
this.axios.get('https://restapi.amap.com/v3/ip?parameters',
               {params:{'key':'应用key'}})
.then(res=>{
           console.log(res.data)
          })
```

执行之后就可以看到 结果了

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

还可以使用python 脚本来请求 

```python
import requests

res= requests.get('https://restapi.amap.com/v3/ip?parameters',params={'key':'c8165322c2c48892f90ccda1aef617cf'})
print(res.json())
```

只需要三行代码就知道自己所在的位置和坐标拉 

岂不是很简单！