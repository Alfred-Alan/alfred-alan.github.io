---
layout: post
title: 'vue实现及分页展示多文件存储'
subtitle: '如何一次保存多个文件'
date: 2020-05-12
categories: 技术
tags: django vue 
image: /assets/img/blog/pagination.png


---

####  分页展示

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

前端使用heyui展示

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

定义需要的变量

```js
pagination:{
    page:1,
    size:4,
    total:0
},
```

发送请求

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



### 多文件上传

在一些开发场景中 是否遇到过 多文件一次性上传

今天尝试了一些新的方法来实现这个功能

思路是  判断input标签有变化之后  从标签中获取文件 以key:value形式存入list

点击提交的时候读取list循环打包 formdata 向后台传递数据

首先监测input标签

```js
add_infoimgs(e){
    // 获取文件
    let file = e.target.files[0];
    //添加进列表
    this.info_imgs.push({
        name:file.name,
        file:file
    })
```

之后是批量封装formdata

```js
//循环列表
for(let i=0;i<this.info_imgs.length;i++){
    // 获取每次循环文件
    let file = this.info_imgs[i].file
    //添加进formdata
    formdata.append(file.name, file, file.name)
    
    //如果同时对一个名称进行添加 就形成一个list
    //添加图片名list
    formdata.append("imgs_name",file.name)
}
```

定义参数类型

发送请求

```js
let config = { headers: { "Content-Type": "multipart/form-data" } };
this.axios.post("http://127.0.0.1:8000/myapp/insertgoods/",formdata,config).then(res=>{
    console.log(res.data)
})
```

django 后台思路  根据图片名list 来判断有没有文件

如果有 循环list次数来取文件

```python
# 获取详情图片的列表
# POST.getlist 可以将参数转换为list
name = request.POST.get('name',None)
imgs_name = request.POST.getlist('imgs_name',None)

 # 如果存在详情图片
    if imgs_name:
        for i in imgs_name:
            myfile = request.FILES.get(i)
            print(myfile.name)
            # 保存详情图
            with open(os.path.join(UPLOAD_ROOT, '', myfile.name), 'wb')as f:
                for chunk in myfile.chunks():
                    f.write(chunk)
        # 将关系存入 mongodb
        # 在展示的时候就可以使用name 来查询相关图片信息
        table.insert({'name': name, 'imgs': imgs_name})
```

使用

```python
info_img=table.find_one({"name":name},{'imgs':1})['imgs']
```

