---
layout: post
title: '分页底层实现及排序查找'
description: '如何对过多的分页按钮进行省略'
categories: [Python,Vue]
tags: [Django] 
image: /assets/img/blog/Pagination-plugin.png
---
- Table of Contents
{:toc .large-only}


##  分页展示

概念：
先指定一个展示多少个
总个数除以每页展示多少个 得出有多少页  

model 类获取数据之后 可以使用切片 来指定返回数据  
这样根据 当前页数 就可以指定返回数据

```python
# 当前页
page = int(request.GET.get('page',1))
# 显示个数
size = int(request.GET.get('size',1))

# 计算从哪开始切 因为根据下标开始切 而列表初始下标是0
# 默认的页数是1 所以就要-1
data_start = (page-1)*size

# 计算切到哪 这样切片间隔就是一页的数据
data_end = page * size

# 对结果集切片操作 
goods= Goods.objects.all()[data_start:data_end]
# 查询所有数据个数
count = Goods.objects.count()

# 页数列表
page_list=[]
# 整除除以展示数量
if (count % size)%2==0:
    page_list=[i for i in range(1,count % size+1)]
else:
    # 不能整除 +2
    page_list=[i for i in range(1,count % size+2)]
    
return Response({'data':goods_ser.data,'total':count})
```

## heyui分页按钮

```html
<div class="page_button">
    <Pagination  v-model="pagination" @change="currentChange" layout="pager" small></Pagination>
</div>
<style>
.page_button{
    left: 50%;
    top: 50%;
    margin-left:45%;/*对应中间盒子宽度的一半*/
    margin-top:7%;/*对应中间盒子高度的一半*/

}
</style>
```

## 定义变量

```js
pagination:{
    page:1,
    size:4,
    total:0
},
```

## 发送请求

```js
//获取商品
get_goods(){
    this.axios.get('http://127.0.0.1:8000/myapp/goodslist/?page='+this.pagination.page+'&size='+this.pagination.size).then(res=>{
        this.goods_list=res.data.data
        this.pagelist = res.data.page_list
        this.pagination.total = res.data.total
    })
},
```
## 翻页功能 
分页按钮底层的实现方法  
上一页 下一页 及 省略过多页数  

直接自己手写方法来实现功能  
上一页下一页 及点击翻页

### 定义变量

```js
// 数据列表
goods_list_self:[],
// 数据总和
self_total:0,
//上一页 
lastpage:0,
// 下一页
nextpage:0,
// 当前页数
page:1,
// 所有页数
allpage:0,
// 展示数量
size:1,
```

### 思路：

  + 上一页：当前页数为1时不变，否则 当前页数-1 即上一页
  + 下一页：要先计算出总页数/末页  当前页数等于最后一页时不变，否则 当前页数+1 即下一页
  + 翻    页：在循环输出页数的时候的时候  重新调用并传参 @click绑定 get_goods_self(参数)

发送请求 变量赋值

```js
//封装函数
get_goods_self(page){
    // 传参可复用 每次翻页将页数付给当前页数
    this.page=page
    this.axios.get('http://127.0.0.1:8000/myapp/goodslist/',{
        params:{page:page,size:this.size}
    }).then(res=>{
        console.log(res.data)
        this.goods_list_self=res.data.data
        this.self_total = res.data.total
        
        //判断上一页
        if(page==1){
            this.lastpage = 0;
        }else{
            this.lastpage = page-1
        }
        
        //计算总页数
        //Match.ceil() 向上取整  除不尽就+1
        this.allpage = Math.ceil(this.self_total / this.size)
        //判断下一页
        if(page == this.allpage){
            this.nextpage = 0
        }else{
            this.nextpage = page +1
        }
```

```js
// 点击调用函数并传参 
<button v-for="index in allpage" @click="get_goods_self(index)">{ {index} }</button>
```

## 如何省略过多的页数

### 思路：

  + 一共有 10 页 ``` 1 2 3 4 5 6 7 8 9 10```
  + 比如此时页数为 5 要省略掉左边和右边过多的页数
  + 展示为 ```...  3 4 5 6 7 ...```
  + 而我们管控制左右显示数量叫为偏移量  即 5 的左边可以显示 3 4 两个按钮
  + 根据中间页数求出两边展示页数
  + 实现：两边都是可变的列表 每次更换中间页数 列表的值重新生成

```js
//设置偏移量 两边可以展示两个
var move_page = 2
```

生成左侧页数列表

```js
var my_last=[]
// 计算左侧偏移量
// 起始位置=当前页-偏移量 例 5-2=3 从3开始展示 
// 不能大于当前页 i<5 生成3 4 
for(let i=page-move_page;i<page;i++){
    // 大于0 左侧只能从1开始
    if(i>0){
        my_last.push(i)
    }
}
this.last_page=my_last  //赋值
```

生成右侧页数列表

```js
var my_next=[]
// 计算右侧偏移量
// 右侧起始要大于当前页 例 5+1=6 从6开始
// 要小于等于当前页+偏移量 例 5+2=7 展示6 7
for(let i=page+1;i<=page+move_page;i++){
    // 小于等于末页 不能超过末页
    if(i<=this.allpage){
        my_next.push(i)
    }
}
this.next_page = my_next
```

## 网页展示

左侧页数列表 +  当前页数 + 右侧页数列表

```html
<button v-for="index in last_page" @click="get_goods_self(index)">{ {index} }</button>
<button  @click="get_goods_self(page)">{ {page} }</button>
<button  v-for="index in next_page" @click="get_goods_self(index)">{ {index} }</button>
```



## 排序查找

django后台 model类可以order_by 排序

```python
# 排序字段
column = request.GET.get('column',None)
# 顺序 默认空是正序
sort_order = request.GET.get('order',"")
```

### 查询操作

```python
# 如果有排序字段
if column:
    goods= Goods.objects.all().order_by(sort_order+column) # 字符串拼接
else:
    goods= Goods.objects.all()
```


### 前端定义两个变量

```js
//排序变量
column:'',
order:'',
```

### 默认带空参数

```js
get_goods_self(){
    this.axios.get('http://127.0.0.1:8000/myapp/goodslist/',{
        params:{column:this.column,order:this.order}
    }).then(res=>{
        console.log(res.data)
```

### 当点击按钮的时候对排序变量赋值

```html
<Button @click="orderby('price')">按价格排序</Button>
<Button @click="orderby('create_time')">按日期排序</Button>
 <!--django后台 -号为倒序-->
<Button @click="sort('')">↑</Button> / <Button @click="sort('-')">↓</Button>
```

### 添加点击事件

```js
//排序操作
orderby(column){
    // 接受参数赋值
    this.column=column
    // 重启执行请求函数
    this.get_goods_self()
},
sort(order){
    this.order = order
    // 重启执行请求函数
    this.get_goods_self()
},
```

