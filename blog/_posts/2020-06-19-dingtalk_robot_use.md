---
layout: post
title: '如何使用钉钉群机器人推送消息'
subtitle: '怎样注册使用机器人'
date: 2020-06-19
categories: 技术
tags: dingtalk
image: /assets/img/blog/dingding.png

---

随着疫情而来，很多公司依靠阿里旗下的办公软件钉钉来进行远程办公，

钉钉在上半年可谓是大火了一把，可谓是有人喜有人忧。

而钉钉内部的机器人功能还是很实用的，它可以实现信息的自动化同步，例如：通过聚合Github、Gitlab等源码管理服务，实现源码更新同步；通过聚合Trello、JIRA等项目协调服务，实现项目信息同步；同时，支持Webhook协议的自定义接入，支持更多可能性。

可以使用机器人来推送自己定义的一些信息，比如打卡，定时课程，或者是定时发布天气。

但钉钉机器人开发文档 都是基于python 2  来配置，今天试着用python3 来配置钉钉机器人。

 首先明确一点，钉钉自定义机器人早就不支持在手机端创建了，所以打开你的pc端或者mac端的钉钉客户端，在需要机器人的聊天群界面，点击智能群助手。

![选择助手](/assets/img/dingtalk/选择助手.png)

<br/>

之后点击创建机器人

![点击机器人](/assets/img/dingtalk/点击机器人.png)

<br/>

  此时能看到很多已经封装好的第三方机器人，本次我们选择自定义机器人

![创建机器人](/assets/img/dingtalk/创建机器人.png)

<br/>

  值得一提的是，钉钉的机器人基于webhook协议，webhook呢是一个api概念,是微服务api的使用范式之一,也被成为反向api,即前端不主动发送请求,完全由后端推送，有机会会单门写一篇文章阐述webhook

  在添加机器人界面里，填写一些机器人的信息

![选择加密方式](/assets/img/dingtalk/选择加密方式.png)

<br/>

  需要注意的是，在安全设置一栏里，我们选择加签的方式来验证，在此说明一下，钉钉机器人的安全策略有三种，第一种是使用关键字，就是说你推送的消息里必须包含你创建机器人时定义的关键字，如果不包含就推送不了消息，第二种就是使用加密签名，第三种是定义几个ip源，非这些源的请求会被拒绝，综合来看还是第二种又安全又灵活。

![选择webhook](/assets/img/dingtalk/选择webhook.png)

<br/>

现在我们就拿到了机器人的签名，和他的webhook地址

查看官方文档：https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq 

接下来上代码

```python
import time
import hmac
import hashlib
import base64
import urllib.parse

timestamp = str(round(time.time() * 1000))
secret = 'SEC90485937c351bfaed41fea8eda5f1e155bbf22842d5f9d6871999e05822fd894'
secret_enc = secret.encode('utf-8')
string_to_sign = '{}\n{}'.format(timestamp, secret)
string_to_sign_enc = string_to_sign.encode('utf-8')
hmac_code = hmac.new(secret_enc, string_to_sign_enc, digestmod=hashlib.sha256).digest()
sign = urllib.parse.quote(base64.b64encode(hmac_code))
# print(timestamp)
# print(sign)


import requests,json   #导入依赖库
headers={'Content-Type': 'application/json'}   #定义数据类型
webhook = 'https://oapi.dingtalk.com/robot/send?access_token=f0ca7636f5812fe4815c97a72de9a7cc780c414c258b6c9a631036b1d0f49e3b&timestamp='+timestamp+"&sign="+sign
#定义要发送的数据
#"at": {"atMobiles": "['"+ mobile + "']"
data = {
    "msgtype": "text",
    "text": {"content": '都谁没加到群里来？小心升不了班'},
    "isAtAll": True}
res = requests.post(webhook, data=json.dumps(data), headers=headers)   #发送post请求

print(res.text)
```

当返回值无异常的时候，就可以查看钉钉群内的机器人有没有发消息

参考文档 https://v3u.cn/a_id_132

