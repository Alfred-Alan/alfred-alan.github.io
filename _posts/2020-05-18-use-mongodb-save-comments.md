---
layout: post
title: '实现商品评论功能'
description: '如何使用mongodb进行存储评论'
categories: [Python]
tags: [Django]
image: /assets/img/blog/mongodb.jpg

---


- Table of Contents
{:toc .large-only}

## 说到评论

一些大型商品网站都会有评论功能 从而看出商品的质量

我们就来做一个类似的评论功能

首先使用``heyui`` 提供的模块 来渲染成一个输入框

```html
<!-- 商品评论 -->
<!--v-wordcount 控制字数-->
<textarea rows="5" v-model="comment" v-autosize  v-wordcount='100'>

</textarea>
<Button color ='blue' @click="submit">提交评论</Button>
```

## 发送请求
创建函数 使用axios发送请求

```js
// 提交评论
	submit(){
        // 判断评论规格
		if(this.comment==''){
			this.$Message('评论不能为空')
			return false
		}
		this.comment=this.comment.replace(/ /g,"")
		if(this.comment.length>100){
			this.$Message('超过长度限制')
			return false
		}
		// 请求入库
        var username=localStorage.getItem('username')
		var data={
			username:username,
            // gid 为商品id
			gid:this.id,
            // comment 商品评论数据
			content :this.comment}
		// 发送请求
		this.axios.post('http://127.0.0.1:8000/myapp/commentinsert/',data).then(res=>{
			// 为了节省资源 评论成功后 直接对原来展示的数据 添加假数据
			this.comments.unshift({'username':username,'content':this.comment,'gid':this.id})
			this.$Message(res.data.msg)
		})

	},
```

## 评论入库功能

为了防止恶意刷评论 限制ip在30s 内只能评论3次

使用redis中的list数据类型 当评论成功时 向列表添加1 并设置过期时间

如果多次添加就会在短时间内 大于3 从而被拦截

当该列表过期后才可以继续添加

```python
def post(self,request):
    
    # 获取客户端ip
    if 'HTTP_X_FORWARDED_FOR' in request.META:
        ip = request.META.get('HTTP_X_FORWARDED_FOR')
    else:
        ip= request.META.get('REMOTE_ADDR')
        print(ip)
        # 30 秒内只能评论三次
        
    try: # 判断该ip列表大于3
        if r.llen(ip)>3 :
            return Response({'code':403,'msg':'您评论过快'})
        except Exception as e:
            pass

    try:
        self.table.insert({
            'username': request.POST.get('username',None),
            'gid':request.POST.get('gid'),
            'content':request.POST.get('content'),
        })
        r.lpush(ip, 1)
        r.expire(ip, 30)
    except :
        pass

   return Response({'code':200,'msg':'评论成功'})
```





## django后端展示评论

```python
def get(self,reuqest):
    gid = reuqest.GET.get('gid',None)
    print(gid)
    # 使用mongodb 查询数据 以id为倒序输出
    comments = self.table.find({'gid':gid},{'_id':0}).sort([("_id",pymongo.DESCENDING)])

    return Response({'code':200,'data':list(comments)})
```

## 获取评论数据接口

```js
//获取评论
get_comment(){
    this.axios.get('http://127.0.0.1:8000/myapp/commentinsert/',{
        // gid 为商品id
        params:{gid:this.id}
    }).then(res=>{
        this.comments=res.data.data
    })
},
```