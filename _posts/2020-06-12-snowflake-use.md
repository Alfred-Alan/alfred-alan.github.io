---
layout: post
title: '什么是SnowFlake算法，又用在什么场景'
description: '如何使用SnowFlake,在什么情况下使用'
image: /assets/img/blog/steve-harvey.jpg


---

***什么是SnowFlake：***

SnowFlake 其翻译过来是雪花，顾名思义 因为每片雪花的样式形状都不同，每一片雪花都是一个独立存在的个体。

而SnowFlak算法就是拥有其特性，使用该算法生成的id永远不会重复。

并且是随着时间而递增，同时还是19位整形数字，适合mysql中的bigint类型，正因为他是独立的并且自增还可以作为表中主键的存在。

到这里有人会说, 要是生成独立的一串id 我使用UUID可以吗

因为UUID 生成的字符串是由64位数字字母组成的字符串，占用空间大且不说，因为他是使用数字字母组成，可读性非常差，而且还是无序的，里面也有字母也就别指定自增了。

但SnowFlake是基于二进制来生成的id 我们还可以对他进行反解析 将它转成二进制，按照所需来截取一段二进制码进行解析

![bitcode](/assets/img/SnowFlake/bitcode.jpg)

1位标识符 为0

41位时间截(毫秒级)，注意，41位时间截不是存储当前时间的时间截，而是存储时间截的差值（当前时间截 - 开始时间截) 后得到的值，这里的的开始时间截，一般是我们的id生成器开始使用的时间，由我们程序来指定的

10位的数据机器位，可以部署在1024个节点，如果分布部署的话 这10位是由5位数据中心ID + 5位机器ID 组成

12位序列，毫秒内的计数，12位的计数顺序号支持每个节点每毫秒(同一机器，同一时间截)产生4096个ID序号

<br/>

***在什么场景使用：***

第一个使用该算法的公司是国外的Twitter，因为Twitter是面向国际化其用户也是非常多。

每个用户都可以发布自己的文章，并且都可以发布评论。

这就造成的数据量过大，一台机器搭载的数据库完全承受不住

一台不行 那就多来几台，使用分布式系统来存储数据。每一台机器内都有数据库，可以向不同的数据库内写入信息

那么问题就在这。我怎么知道我这条帖子是保存在哪台机器中，该从哪个数据库中读取？

Twitter就提出了Snowflake(雪花算法) 在不同的机器下，同一个时间内所生成的id也是不同的，并且生成的id还可以反解析

通过这个id找到我是从哪个机器组成 从而找到数据库的位置

非常的适用于分布式存储系统

<br/>

***如何使用：***

先安装模块

```powershell
pip3 install pysnowflake
```

安装之后就可以启动服务

```powershell
snowflake_start_server --worker=1 --port=8910
```

使用get_stats()就可以打印出当前客户端使用的snowflake服务信息

```python
import  snowflake.client

print(snowflake.client.get_stats())

{'dc': 0, 'worker': 1, 'timestamp': 1592114075483, 'last_timestamp': 550281600000, 'sequence': 0, 'sequence_overload': 0, 'errors': 0}
```

如果一台机器开启了多个服务，还可以主动指定连接

```python
import  snowflake.client
# 可以主动连接自指定端口
snowflake.client.setup('localhost',8910)
```

现在就可以使用生成该服务生成id了

```python
print(snowflake.client.get_guid())
4369762851619868673
```



***如何解析：***

通过十进制的19位id 解析为二进制64位编码

```python
print(bin(4369762851619868673))
0b11110010100100100001000010100101011011100000000001000000000001
```

而我们仔细看的话就与上面的规律图相似

```powershell
0   b1111001010010010000100001010010101101110   00000     00001  000000000001
标识|		          41位时间戳       | 数据中心id | 机器id| 序列号 
```

拿到机器id 转换为10进制看一下

```python
print(int('00001',2))
1
```

就可以看到当前worker的值为1

这样一来无论是生成id 还是反向解析都是很方便操作的

只需搞懂组合的逻辑 并且熟悉二进制转换，就是小菜一碟了