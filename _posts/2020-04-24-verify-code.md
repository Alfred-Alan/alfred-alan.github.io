---
layout: post
title: 'Python生成验证码'
description: '如何使用Python生成自定义验证码 并进行验证'
categories: [Python]
image: /assets/img/blog/python.png

related_posts:
  - _posts/2020-04-23-django-use.md
  - _posts/2020-04-22-vue-use.md
---

- Table of Contents
{:toc .large-only}

## 怎样使我们自定的字符串变成一个图片验证码

使用python中的图像库来实现

Python图像库PIL(Python Image Library)是python的第三方图像处理库，但是由于其强大的功能与众多的使用人数，几乎已经被认为是python官方图像处理库了。其官方主页为:[PIL](http://pythonware.com/products/pil/)

 Image类是PIL中的核心类，你有很多种方式来对它进行初始化，比如从文件中加载一张图像，处理其他形式的图像，或者是从头创造一张图像等。Image模块操作的基本方法都包含于此模块内。如**open**、**save**、**conver**、**show**…等方法。

## 导入所需要的包

```python
# 文件流
import io
# 随机库
import random
# 导入redis数据库
import redis
# 导入图片库
# 图片库
from PIL import Image
# 绘画库
from PIL import ImageDraw
# 字体库
from PIL import ImageFont
```

## 定义一个类

```python
class MyCode(object):
    # 定义rgb随机颜色
    def get_random_color(self):
        R = random.randrange(255)
        G = random.randrange(255)
        B = random.randrange(255)
        return (R, G, B)

    # 定义图片视图
    def get(self):
        # 画布
        img_size = (120, 50)
        # 定义图片对象
        image = Image.new('RGB', img_size, 'white')
        # 定义画笔
        draw = ImageDraw.Draw(image, 'RGB')
        source = '0123456789abcdefghijk'
        # 接受容器
        code_str = ''
        # 进入循环绘制
        for i in range(4):
            # 获取字母颜色
            text_corlor = self.get_random_color()
            # 获取随机下标
            tmp_num = random.randrange(len(source))
            # 随机字符串
            random_str = source[tmp_num]
            # 装入容器
            code_str += random_str
            # 绘制字符串
            draw.text((10 + 30 * i, 20), random_str, text_corlor)
            # 将验证码存入redis
            r = redis.Redis('localhost', 6379)
            r.set('code', code_str)
            r.close()
        image.save('./test.jpg')
```

## 如何取出验证码验证

```python
# 链接redis
r = redis.Redis('localhost', 6379)
# 判断验证码
if code != r.get('code').decode('utf-8'):
   return '验证码错误'
```

#### 以及如何使用vue来接受图片的二进制流

因为前段接受的数据默认为json形式，如果接受二进制会出现乱码情况，这时候就需要指定接受类型

在Web领域，Blob被定义为包含只读数据的类文件对象。Blob中的数据不一定是js原生数据形式。常见的File接口就继承自Blob，并扩展它用于支持用户系统的本地文件。

```js
axios({
        url:"http://127.0.0.1:8000/myapp/mycode/",
        method:'get',
        responseType: 'blob',  //指定接受的类型
      }).then(res=>{
         this.imgcode  = window.URL.createObjectURL(res.data);
          //src 就是一个可以显示图片的相对路径。因为window.URL.crateObjectURL(blob)已经进行了转换
      })
```

#### 浏览器控制台application的了解

cookie与storage的异同：

相同点：同样受同源策略影响，只有在域名一致的情况下才能查看到对应的数据。

不同点：

1.cookie的数据存储量在4K左右，storage的存储量大约在4MB左右；

2.navigator.cookieEnabled检测是否启用了cookie，也就说cookie可以认为控制是否启用。storage则是自动启用，不会被人为关闭。

3.cookie中不建议使用分号、逗号、空格等特殊字符；如果需要使用可以使用转码操作：

```js
encodeURIComponent()//传入特殊字符转码，可以应用转码作为简单的加密处理
decodeURIComponent()//将转码的字符转换为正常字符
```

storage中只能接收字符串类型的数据，具体操作见下一节。

4.cookie有时效性可以设置有效时间，如果不设置的话在浏览器窗口关闭时就会失效；storage是永久存储。

5.cookie会与服务器通信；storage只存在客服端，不参与服务器通信。

##### Storage的应用

localstorage 永久保存 (不主动删除 则一直存在)

session storage 会话存储临时保存  当浏览器关闭时自动清空

##### 存入或取出

```js
存入：localStorage.setItem('key',value)
取出：localStorage.getItem('key')
清空：localStorage.removeItem('key')
```





