---
layout: post
title: '如何查找当前节点所有父节点'
description: '使用pandas.dataframe递归找出所有父节点'
categories: [Python]
tags: []
image:  /assets/img/blog/dataframe.png

related_posts:
  - _posts/2021-8-18-make-tree.md
---
- Table of Contents
{:toc .large-only}

### 场景
最近在优化查询的时候 发现部门查询要自查很多次
部门表是自关联，这就导致部门批量查询要耗费很长时间

需要通过当前用户的部门 查询所有上级部门

就是从当前节点查询所有父节点

以达到此效果:

![departments](/assets/img/parent_nodes/departments.png)

表结构：

| id   | name  | pid  |
| ---- | :---- | :--: |
| 1    | 1     |  0   |
| 2    | 1-1   |  1   |
| 3    | 1-2   |  1   |
| 4    | 1-2-1 |  3   |
| 5    | 1-2-2 |  3   |
| 6    | 1-2-3 |  3   |
| 7    | 1-3-1 |  2   |

### Pandas.DataFrame

我们需要使用 ``Pandas.DataFrame`` 来处理数据结构

DataFrame是一种表格类型的数据结构，它含有一组有序的列，每列可以是不同的类型的值。DataFrame 既有行索引也有列索引，它可以被看做由 Series 组成的字典（共同用一个索引）

对比Excel就能理解了，A B C D 就是每个series，而series里面的下标就是索引

![Excel](/assets/img/parent_nodes/Excel.png)

DataFrame的创建有多种方式，我们这次主要使用dict来创建。


### 数据结构

查出的数据

```python
department_list = [
    {"id": 1, "name": "1", "pid": 0},
    {"id": 2, "name": "1-1", "pid": 1},
    {"id": 3, "name": "1-2", "pid": 1},
    {"id": 4, "name": "1-2-1", "pid": 3},
    {"id": 5, "name": "1-2-2", "pid": 3},
    {"id": 6, "name": "1-2-3", "pid": 3},
    {"id": 7, "name": "1-3-1", "pid": 2},
]
```

我们可以把id 和pid 看做为两个series

生成`ids`和`pids`两个键的字典结构

```python
department_id_series = {
    'ids': [],
    'pids': []
}

for department in department_list:
    department_id_series["ids"].append(department['id'])
    department_id_series["pids"].append(department['pid'])

# {'ids': [1, 2, 3, 4, 5, 6, 7], 'pids': [0, 1, 1, 3, 3, 3, 2]}   
```

### 实现方法

使用递归函数来求出父节点

它会将符合条件的行加在一起，重新组成一个独立的表格数据

```python
def get_parent_department_ids(data_frame, department_id):
    # 取 data_frame里对应当前id的一行数据
    tmp_df = data_frame[data_frame["ids"] == department_id]

    # 如果当前数据为空 or pid为None
    if tmp_df.empty or tmp_df["pids"].values[0] is None:
        return pd.DataFrame(columns=["ids", "pids"])
    else:
        # 递归查找 pid的frame数据
        # append(tmp_df) 当前data_frame数据追加到递归结果上
        return get_parent_department_ids(data_frame, tmp_df["pids"].values[0]).append(tmp_df)
```

这个递归函数 我测试到900多层才会触发 递归深度

一般不会有组织/菜单 关联到900多层的吧

### 执行结果

```python
data_frame = pd.DataFrame(department_id_series)

# 依次循环子节点的id
for department_id in department_id_series['ids']:
    department_parent_ids = get_parent_department_ids(data_frame, department_id)['pids'].values.tolist()
	print(department_id, department_parent_ids)

# 1 [0]
# 2 [0, 1]
# 3 [0, 1]
# 4 [0, 1, 3]
# 5 [0, 1, 3]
# 6 [0, 1, 3]
# 7 [0, 1, 2]
```

这个时候就可以使用`parent_id`列表去拼装对应的部门数据了

参考：https://blog.csdn.net/chinacmt/article/details/79716847