---

layout: post
title: '使用邮箱发送验证邮件'
description: '如何使用邮箱发送邮件'
categories: [Python]
tags: [Email]
image: /assets/img/blog/email.jpg

related_posts:
  - _posts/2020-05-29-send-sms.md

---

- Table of Contents
{:toc .large-only}

## 发送邮件

在一些注册页面中都有邮箱验证，通过邮箱发送验证码判断注册

今天来实践一下 如何使用邮箱发送邮件

<br/>

## 如何开启服务：

[登录QQ邮箱](https://mail.qq.com/)

找到设置-账户![设置账户](/assets/img/send_mail/set_account.png)

<br/>

选择开启 IMAP/SMTP服务  

![开启服务](/assets/img/send_mail/start_serve.png)

<br/>

点击获取授权码  

![通过验证](/assets/img/send_mail/approved.png)

<br/>

通过验证之后就可以拿到授权码保存  

有了授权码 我们就可以 进行邮件发送了

<br/>

## 如何使用IMAP服务：

+ **接收邮件服务器：**imap.qq.com 
+ **发送邮件服务器：**smtp.qq.com
+ **账户名：**您的QQ邮箱账户名（如果您是VIP邮箱，账户名需要填写完整的邮件地址）
+ **密码：**您的QQ邮箱授权码
+ **电子邮件地址：**您的QQ邮箱的完整邮件地址

<br/>

## 如何设置IMAP服务的SSL加密方式？

+ 使用SSL的通用配置如下：

+ **接收邮件服务器：**imap.qq.com，使用SSL，端口号993

+ **发送邮件服务器：**smtp.qq.com，使用SSL，端口号465或587

+ **账户名：**您的QQ邮箱账户名（如果您是VIP帐号或Foxmail帐号，账户名需要填写完整的邮件地址）

+ **密码：**您的QQ邮箱授权码

+ **电子邮件地址：**您的QQ邮箱的完整邮件地址

 <br/>

## 创建发送邮件函数

```python
import smtplib
from email.mime.text import MIMEText
from email.utils import  formataddr

# 邮箱
my_mail = '3210440292@qq.com'
# 授权码
my_pass = 'dkptmjeotyhzdhch'

def mail(subject,content,mailladdress):
    # 声明邮件对象
    msg = MIMEText(content,'plain','utf-8')
    # 设置发送方对象
    msg['From']= formataddr(['在线教育平台',my_mail])
    # 设置收件方对象
    msg['To'] = formataddr(['尊敬的客户',mailladdress])
    # 设置标题
    msg['Subject'] = subject
    # 设置smtp服务器
    server = smtplib.SMTP_SSL(host='smtp.qq.com',port=465)
    # 登录邮箱
    server.login(my_mail,my_pass)
    # 发送文件
    server.sendmail(my_mail,[mailladdress],msg.as_string())
    # 关闭smtp链接
    server.quit()
    
    
mail('注册验证码','2020','****@qq.com')
```

