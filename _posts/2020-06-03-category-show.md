---
layout: post
title: '实现展示无限极分类'
description: '如何实现展示无限极分类'
categories: [Python,Vue]
tags: [Django]
image: /assets/img/blog/category.jpg

related_posts:
  - _posts/2020-04-23-django-use.md

---
- Table of Contents
{:toc .large-only}

## 无限极分类
无限极分类功能就相似于淘宝的分类模块。  

生活用品--衣服--夏季衣服--男士 像这一类的分类。  

正常思考是外键来关联每个数据的关系，但如何使用一张表来实现这个功能呢。  

可以对数据添加一个伪外键 例：pid 为0是顶级父类  其指向的是父类的id

```python
[{'id': 1, 'name': '在线课程', 'pid': 0},{'id': 2, 'name': 'test', 'pid': 1}]
```

从数据库查出时 是每个独立的一条数据

我们要制作一个多层级的数据结构

列表中可以有 字典 字典的键还可以嵌套列表

```python
{'id': 1, 'name': '在线课程', 'pid': 0, 'child': [{'id': 2, 'name': 'python', 'pid': 1}]}
```
## 思路

使用递归思想来制作多层级结构
从列表的尾部 向前每一个元素的递归
当现在下标上的元素 没有的时候 返回 数据列表

```python
# file: 'utils.py'

tree = {}
lists = []
def test(alist):
    for item in alist:
        tree[item['id']] = item

    # 根据长度 获取尾部下标
    n = len(alist)-1
    if n==-1:
        return lists

    if not alist[n]['pid']:  # 如果没有父类 正常添加
        lists.append(tree[alist[n]['id']])
    else:
        pid =  alist[n]['pid']
        # 判断他的父类 有没有child键
        if "child" not in tree[pid]:
            tree[pid]["child"] = []  # 创建child键
        # 将当前子类填充到父类child键里
        tree[pid]['child'].append(tree[ alist[n]['id']])
	
    return atest(alist[:n]) #最后元素已经添加 切片到前一个元素
```

## 执行结果

```python
[{"id": 1,"name": "python","pid": 0,"childlist": [{"id": 4,"name": "爬虫","pid": 1 }]},
 {"id": 2,"name": "java","pid": 0},
 {"id": 3,"name": "html", "pid": 0}]
```

通过这个函数就可以对查询出的结果进行操作

```python
# file: 'views.py'
def get(self,request):
    categorys=Category.objects.all()
    categoryser= Category_ser(categorys,many=True)

    return Response({'code':200,'data':test(categoryser.data)})
```

## 后端返回结果

```json
{
    "code": 200,
    "data": [
        {
            "id": 1,
            "name": "python",
            "pid": 0,
            "childlist": [
                {
                    "id": 4,
                    "name": "爬虫",
                    "pid": 1
                }
            ]
        },
        {
            "id": 2,
            "name": "java",
            "pid": 0
        },
        {
            "id": 3,
            "name": "html",
            "pid": 0
        }
    ]
}
```
## Vue自定义组件
在vue 中 递归展示分类可以自定义一个组件

给组件传值后判断 可以再次在组件中自调用

新建一个```Reply.vue``` 文件

```vue
<!--file: 'Reply.vue'-->
<template>
  <div>
        <li > 
            <!--输出当前data.name -->
            <div >{{ data.name }}</div>
            <!--如果当前data.childlist 存在-->
            <ul v-if="data.childlist && data.childlist.length>0">
                <Reply v-for="child in data.childlist" :key="child.id" :data="child"/>
            </ul>
        </li>

  </div>
</template>

<script>
export default {
    name: 'Reply', // 递归组件需要设置 name 属性，才能在模板中调用自己
    props:['data'], // 接受参数为 data
 }
</script>

<style>
.reply {
    padding-left: 4px;
    border-left: 1px solid #eee;
}

ul {
    padding-left: 20px;
    list-style: none;
}

.root { display: none; }

.myfooter{
    position:absolute;
    bottom:0;
    width:100%;
}
</style>
```
## 导入自定义组件
在展示页面 导入 Reply.vue 并且使用

```vue
<!--file: 'menu.vue'-->
<template>
  <div>
        <div class="form">
            <ul>
                <p v-for="data in datas">
                  	<Reply  :data="data" />
                </p>
            </ul>
        </div>
  </div>
</template>

<script>
import Reply from './Reply.vue';
import axios from 'axios'
export default {
    data(){
        return{ 
          datas:{},
          online: 0
        }
    },
    components:{
        'Reply':Reply,
    },
    created(){
        this.get_category()
    },
    methods:{
        get_category(){
            axios.get('http://127.0.0.1:8000/myapp/show_category/').then(res =>{
                this.datas = res.data.data;
            });
        },
    }
}
</script>

<style>

</style>
```

## 执行结果

* python
	+ 爬虫
* java
* html

