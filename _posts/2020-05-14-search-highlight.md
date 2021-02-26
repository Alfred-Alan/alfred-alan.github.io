---
layout: post
title: '多条件模糊查询及关键字高亮'
description: '如何对查询的关键字高亮显示'
categories: [Python,Vue]
tags: [Django]
image: /assets/img/blog/search.png
---

- Table of Contents
{:toc .large-only}

## 如何实现多条件模糊查询 

在我们一般使用淘宝网站的时候  
经常使用一些连词  来精准查询的结果

例：使用 ```夏季  鞋子 男 休闲``` 等词语来指定查找  

那么如何实现该功能呢

```html
<!--实现搜索框-->
<input @change='search' v-model="text" >
```
## 解析条件
text 变量就是绑定的搜索的参数

我们只需要判断该变量就可以了

```js
	// 查询字符串中是否有空格
if (this.text.indexOf(' ')){
    //由空格为间隔切片为list
    var text = this.text.split(" ")
    // 生成list形式的字符串
    text = JSON.stringify(text)
}
```
## 获取参数

对查询参数进行操作 生成字符串：["value1","value2"]

在后台接受的参数时将该字符串转换为列表

```python
# 检索字段
text = eval(request.GET.get('text',None))
```

## 模糊查询操作

操作思路：循环查询参数 每次查询之后添加进列表

```python
# 是否进行模糊查询
if text:
    goods=[]
    # 循环条件列表
    for key in text:
        # 每次循环 查询数据
        good_obj=Goods.objects.filter(Q(name__contains=key)|Q(desc__contains=key)).all()
        # 向列表尾部拼接
        goods.extend(good_obj)
        # 去重 防止重复 有时重复查询同一个 会出现多数据 就需要去重
        goods=list(set(goods))
        count = len(goods)
else:
            # 查询所有商品个数
     count = Goods.objects.count()
```



## 关键字高亮

在我们访问百度的时候 经常看见自己查询的关键字是红色的   

这是怎么做到的呢

可以用过滤器来操作

```js
//过滤器
 filters:{
	 make_text(str){
		var mytext=str.toString()
		var text='关键字'
        // new RegExp(text,'g') 正则模式全文检索 
		return mytext.replace(new RegExp(text,'g'),'<span class="highlight">'+text+'</span>')
	 }
 },
```

可以实现为 ```<span class="mystyle">关键字<span> 词语```

这样我们就可以设置指定样式来输出了

使用 v-html 来输出

```html
<!--因为v-html与其他不同 需要以调用方式来使用过滤器-->
<!--如果全局声明了装饰器就不需要 否则就要以$options.filters.装饰器 来使用-->
<span v-html='$options.filters.make_text(item.name)'></span>
```

