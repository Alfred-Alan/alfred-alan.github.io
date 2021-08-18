---
layout: post
title: '如何将一维数组转为树形嵌套结构'
description: '在Python中怎么将一维数组转为树形嵌套结构'
categories: [Python]
tags: []
image:  /assets/img/blog/python.png

related_posts:
  - 
---
- Table of Contents
{:toc .large-only}

在开发中 我们难免会遇到树形结构 但是对于怎样生成树形结构是个问题。

如何将一维数组 组装成树形的嵌套结构？

### 数据存储

众所周知 对于菜单数据存储的表一般都是用的自关联结构 表内数据由一个字段相互关联

我设置自关联的字段设为 `pid` 意为 `parent_id`

使用django orm查询后的数据是这样的

```python
menu_list = [
    {'id': 1, 'name': '在线课程', 'pid': 0},
    {'id': 2, 'name': 'python', 'pid': 1},
    {'id': 3, 'name': '语文', 'pid': 0},
    {'id': 4, 'name': '爬虫', 'pid': 2},
    {'id': 5, 'name': '古诗', 'pid': 3}
]
```

可见每条菜单数据都有关联的`pid`字段，`pid`为0的是一级菜单。

### 封装树形结构

我们在函数中需要定义两个变量

```python
result = [] # 返回的数据
key_map = {} # id字典
```

需要把数组结构转为id对应数据的字典类型

```python
for item in data:
    key_map[item['id']] = item
print(key_map)
# {1: {'id': 1, 'name': '在线课程', 'pid': 0}, 2: {'id': 2, 'name': 'python', 'pid': 1}...}
```

接下来才是重中之重

先循环菜单列表

```python
for i in data:
```

判断 `i`的`pid`是否为0

```python
	# 判断pid 是0 为一级菜单
    if not i['pid']:
        result.append(key_map[i['id']])
```

如果存在`pid`当前为子菜单 需要找到它的父级菜单

```python
	 else:
        pid = i['pid']

        # 判断他的父类 没有children键
        if "children" not in key_map[pid]:
            key_map[pid]["children"] = []  # 初始化children为list

        # 将当前子类填充到父类child里
        key_map[pid]['children'].append(key_map[i['id']])
```

这里 `key_map[pid]` 就是父类的数据 我们把子菜单放进父类的`children`就是`key_map[pid]["children"]`

完整代码：


```python
def make_tree(data):
    result = []
    key_map = {}
    for item in data:
        key_map[item['id']] = item

    for i in data:
        # 判断pid 是0 就正常添加进去
        if not i['pid']:
            result.append(key_map[i['id']])
        # 子分类
        else:
            pid = i['pid']
            # 判断他的父类 有没有child键
            if "child" not in key_map[pid]:
                key_map[pid]["child"] = []  # 创建child键
            # 将当前子类填充到父类child里
            key_map[pid]['child'].append(key_map[i['id']])
    return result
```

结果：

```python
[
    {
        'id': 1,
        'name': '在线课程',
        'pid': 0,
        'children': [{
            'id': 2,
            'name': 'python',
            'pid': 1,
            'children': [{
                'id': 4,
                'name': '爬虫',
                'pid': 2
            }]
        }]
    },
    {
        'id': 3,
        'name': '语文',
        'pid': 0,
        'children': [{
            'id': 5,
            'name': '古诗',
            'pid': 3
        }]
    }
]
```

这里其实是使用了浅拷贝的特性 因为字典是可变类型 就算`id=1`的字典被放进了列表

我对该字典新增键，列表里的字典也会多一个key。

### 展示树形结构

我这边用的是[element](https://element.eleme.cn/#/zh-CN/component)组件的 `el-tree`

```python
<el-tree
    :data="data"
    :props="defaultProps"
>
</el-tree>

<script>
  export default {
    data() {
      return {
        data: [],
        defaultProps: {
          children: 'children',
          label: 'name'
        }
      };
    },
    methods: {
    }
  };
</script>
```

这下大工搞成

