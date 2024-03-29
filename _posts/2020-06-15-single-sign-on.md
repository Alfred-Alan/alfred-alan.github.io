---
layout: post
title: '谈一谈单点登录'
description: '什么是单点登录'
image: /assets/img/blog/sso.png


# accent_image: /assets/img/blog/sso.png
# invert_sidebar: true
---

***什么是单点登录：***

单点登录的英文名叫做：Single Sign On (SSO)

在以前的设计的时候，我们一般把所有的功能都集合到一个系统之上，这样称之为单系统

后来，为了降低每个功能的耦合度，将每个功能拆分到各个系统中。

业内周知的就是阿里的淘宝和天猫了，当你登录的淘宝之后在访问天猫就会自动登录，很显然两个系统都知道了用户已登录。

简单来说，单点登录就是在多系统中，用户只需登录一次，各个系统就知道这个用户已登录

而随之而来的问题就是多个系统之间该如何知道这个用户已登录

<br/>

***多系统登录的问题：***

我们正常的登录逻辑就是在用户登录成功之后，将身份存储于cookie/session，这样网站就可以通过session/cookie知晓用户已经登录

但问题是不同的系统 它的webstorage不会共享，多个系统之间就无法进行沟通

解决方法就是可以把 登录功能单独拿出来，做成一个子系统。

当其他的系统需要登录的时候，都跳转到这个登录系统中，登录成功后由它统一分发token(令牌)， 成功后跳转回原来的页面，并把分发的令牌存储到自己的webstorage里。

当每一次需要登录的时候，都会携带这个token去登录系统中验证，验证他登录是否失效或者未登录。

如果验证成功 则不需要登录，直接回调到来时的页面

<br/>

#### 小结：

如果把单点登录反过来看的话，就会发现与第三方登录逻辑相似。可以把第三方登录平台认为一个登录子系统

那么中间的逻辑就会清晰许多。

同时在登录子系统后端也可以 把用户信息存入redis，以token为键存储，别的系统拿到token就可以直接拿到该用户的数据。



