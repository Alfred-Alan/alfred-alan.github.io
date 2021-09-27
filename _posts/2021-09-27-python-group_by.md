---
layout: post
title: '在python中如何对数组进行分组'
description: '使用itertools.group对数据进行分组'
categories: [Python]
tags: []
image:  /assets/img/blog/groupby.png

related_posts:
  - 
---

- Table of Contents
{:toc .large-only}


### 数据源

一般我们从数据库中查到的数据是这样的

```python
user_list = [
    {"name": "张三", "age": 16, "grade": 1},
    {"name": "李四", "age": 17, "grade": 2},
    {"name": "王五", "age": 16, "grade": 1},
    {"name": "赵六", "age": 18, "grade": 3},
]
```

因为grade年级是关联字段 我们可以拿到关联id

但是我们如何对他们进行分组呢

使用itertools的 groupby

### groupby

groupby 的作用是将可迭代对象中的相邻重复元素整理出来放在一起

使用方法：

```python
from itertools import groupby

for key, items in groupby(["A", "A", "b", "C", "C"]):
    print(key, list(items))
    

# A ['A', 'A']
# b ['b']
# C ['C', 'C']
```

groupby 的挑选规则是由函数完成的，默认是比较两个元素返回的值相等，这两个元素就被认为是在一组的，而函数返回值作为组的key。



但是仅对单个元素进行比对是不行的，因为我们这个是列表套字典的结构。

所以要自定义groupby的规则函数



### 自定义规则

使用operator模块中的itemgetter()函数，作用是获取对象指定域中的值

使用方法：

```python
from operator import itemgetter

a = {"name": "hello", "age": 16}
getter = itemgetter("name")
print(getter(a))


# 结果
# hello
```

使用itemgetter 作为分组规则，这样就可以实现指定字段分组了



### 具体实现

因为groupby 是将相邻重复元素进行分组

所以要先排序好

```python
# 分组依据
user_group_by_getter = itemgetter("grade")

# 排序
user_list.sort(key=user_group_by_getter)

# 分组后的角色列表
user_garde_list = groupby(user_list, user_group_by_getter)

# 年级ID和角色列表的映射 k: garde_id v: user_list

garde_id_user_map = {}
for garde_id, users in user_garde_list:
    garde_id_user_map[garde_id] = list(users)
print(garde_id_user_map)


# 结果
#{
#    1: [{'name': '张三', 'age': 16, 'grade': 1}, {'name': '王五', 'age': 16, 'grade': 1}],
#    2: [{'name': '李四', 'age': 17, 'grade': 2}],
#    3: [{'name': '赵六', 'age': 18, 'grade': 3}]
#}
```

可见group_by 帮你将每个年级分好组

我们使用年级id 就可以匹配到每个年级有多少人



