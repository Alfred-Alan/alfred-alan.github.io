---
layout: post
title: 'Django中间件的使用与生命周期'
subtitle: '使用中间件拦截请求'
date: 2020-05-08
categories: 技术
tags: 
image: /assets/img/blog/middleware.png

---

#### 什么是中间件

中间件就是在**目标**和**结果**之间进行的额外处理过程，在Django中就是**request和response之间进行的处理**，相对来说实现起来比较简单，但是要注意它是对全局有效的，可以在全局范围内改变输入和输出结果，因此需要谨慎使用，

#### 中间件有什么用

如果想要修改**HttpRequest**或者**HttpResponse**，就可以通过中间件来实现。

- **登陆认证**：在中间件中加入登陆认证，所有请求就自动拥有登陆认证，如果需要放开部分路由，只需要特殊处理就可以了。
- **流量统计**：可以针对一些渲染页面统计访问流量。
- **恶意请求拦截**：统计IP请求次数，可以进行频次限制或者封禁IP。

#### 中间件的执行流程

![中间件流程](/assets/img/dya13/中间件流程.png)

#### 注册中间件

在django的settings中注册中间件

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware', 
    'django.middleware.common.CommonMiddleware',
    #'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    #加载自定义中间件   文件夹名.文件名.类名
    'myapp.md_user.MyMiddleware'
]
```

中间件包含五个方法

```python
# 拦截所有请求
process_request(self,request)
# 拦截对视图请求
process_view(self, request, callback, callback_args, callback_kwargs)
# 拦截模板响应数据
process_template_response(self,request,response)
# 拦截异常
process_exception(self, request, exception)
# 拦截响应数据
process_response(self, request, response)
```

自定义中间件

```python
# 导包
from django.utils.deprecation import MiddlewareMixin
#自定义中间件
class MyMiddleware(MiddlewareMixin):
    def process_request(self,request):
        print('过滤中间件')
         # 获取路由 指定路由进行拦截
        if request.path_info.startswith('/myapp/upload/'):
            # 在每次请求的时候验证传递过来的jwt
            uid=request.GET.get('uid',None)
            myjwt=request.GET.get('jwt',None)
            try:
                decode_jwt = jwt.decode(myjwt, '201528', algorithms=['HS256'])
            except Exception as e:
                return Response({'code': 401, 'msg': '您的秘钥已失效'})

            if int(uid)!= int(decode_jwt['data']['uid']):
                return Response({'code':401,'msg':'您的秘钥无权限'})
        pass
    def process_view(self,request,view_func,view_args,view_kwargs):
        pass
    def process_exception(self,request,exception):
        pass
    def process_response(self,request,response):
        # 拦截响应后 要返回响应数据
        return response
```



