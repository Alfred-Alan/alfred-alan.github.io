---
layout: post
title: '利用docker实现Elasticsearch全文检索部署'
subtitle: '如何利用docker实现Elasticsearch全文检索部署'
date: 2020-07-23
categories: 技术
tags: docker Elasticsearch
image: /assets/img/blog/ElasticSearch.jpg

---

Elasticsearch是什么

<br>

Elasticsearch 是一个分布式可扩展的实时搜索和分析引擎,一个建立在全文搜索引擎 Apache Lucene(TM) 基础上的搜索引擎.当然 Elasticsearch 并不仅仅是 Lucene 那么简单，它不仅包括了全文搜索功能，还可以进行以下工作:

+ 分布式实时文件存储，并将每一个字段都编入索引，使其可以被搜索。
+ 实时分析的分布式搜索引擎。
+ 可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据。

<br>

基本概念
先说Elasticsearch的文件存储，Elasticsearch是面向文档型数据库，一条数据在这里就是一个文档，用JSON作为文档序列化的格式，比如下面这条用户数据：

<br>

{
    "name" :     "John",
    "sex" :      "Male",
    "age" :      25,
    "birthDate": "1990/05/01",
    "about" :    "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}

<br>

用Mysql这样的数据库存储就会容易想到建立一张User表，有balabala的字段等，在Elasticsearch里这就是一个文档，当然这个文档会属于一个User的类型，各种各样的类型存在于一个索引当中。这里有一份简易的将Elasticsearch和关系型数据术语对照表:

<br>

关系数据库     ⇒ 数据库 ⇒ 表    ⇒ 行    ⇒ 列(Columns)

Elasticsearch  ⇒ 索引(Index)   ⇒ 类型(type)  ⇒ 文档(Docments)  ⇒ 字段(Fields) 

<br>

介绍如何使用docker来部署 Elasticsearch

首先使用docker 下载 Elasticsearch镜像
这里我使用的是7.0以上的版本，

```powershell
docker pull elasticsearch:7.2.0
```

随后运行Elasticsearch镜像

```powershell
docker run --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=singl
```

容器别名我们就用缩写es来替代，通过 9200 端口并使用 Elasticsearch 的原生 传输 协议和集群交互。集群中的节点通过端口 9300 彼此通信。如果这个端口没有打开，节点将无法形成一个集群，运行模式先走单节点模式。

启动成功之后可以使用浏览器访问一下 ``localhost:9200``

![es](../../../../../assets/img/Elasticsearch/es.png)

<br>

然后将配置复制到主机内

```powershell
docker cp 容器id:/usr/share/elasticsearch/config/elasticsearch.yml ./elasticsearch.yml
```

  打开elasticsearch.yml，可以自己加一些配置，比如允许跨域访问，这样你这台Elasticsearch就可以被别的服务器访问了，这是微服务全文检索系统架构的第一步。

```powershell
vim elasticsearch.yml
```

```powershell
cluster.name: "docker-cluster"
network.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
```

然后停止容器 并且删除他 

```powershell
docker stop es
docker rm es
```

再次启动 Elasticsearch 服务器，而这次需要 使用 -v 挂载命令  将我们配置好的设置文件 挂载到容器中去

```powershell
docker run --name es -v home/bywlop/file/es/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -d elasticsearch:7.2.0
```

挂载这个地方需要注意 ``/es/elasticsearch.yml``

我使用相对路径挂载启动会报错 

建议使用绝对路径来挂载启动 

 另外还有一个需要注意的点，就是Elasticsearch存储数据也可以通过-v命令挂载出来，如果不对数据进行挂载，当容器被停止或者删除，数据也会不复存在，所以挂载后存储在宿主机会比较好一点，命令是：

```powershell
docker run --name es -v home/bywlop/file/es/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v home/bywlop/file/es/data:/usr/share/elasticsearch/data -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -d elasticsearch:7.2.0
```

而仅仅启动还不够

我们需要使用它 

这里我使用Python 来进行操作 

安装依赖的库。

```powershell
pip3 install elasticsearch
```

  新建es_test.py测试脚本

  建立Elasticsearch的检索实例

```python
from elasticsearch import Elasticsearch es = Elasticsearch(hosts=[{"host":'Docker容器所在的ip', "port": 9200}])
```

  这里的host指容器ip，因为可以扩展集群，所以是一个list，需要注意一点，如果是Win10就是系统分配的那个ip,Centos或者Mac os直接写127.0.0.1即可。

  建立索引(Index)，这里我们创建一个名为 article 的索引

```python
result = es.indices.create(index='article', ignore=400)
print(result) 
{'acknowledged': True, 'shards_acknowledged': True, 'index': 'article'} 
```

  其中的 acknowledged 字段表示创建操作执行成功。

  删除索引也是类似的，代码如下：

```python
result = es.indices.delete(index='article', ignore=[400, 404])
print(result)
{'acknowledged': True}
```

  插入数据，Elasticsearch 就像 MongoDB 一样，在插入数据的时候可以直接插入结构化字典数据，插入数据可以调用 index() 方法，这里索引和数据是强关联的，所以插入时需要指定之前建立好的索引。

```python
data = {'title': '我在北京学习人工智能', 'url': 'http://123.com','content':"在北京学习"}
result = es.index(index='article',body=data)
print(result)
{'_index': 'article', '_type': '_doc', '_id': 'GyJgb3MBuQaE6wYOApTh', '_version': 1, 'result': 'created', '_shards': {'total': 2, 'successful': 1, 'failed': 0}, '_seq_no': 5, '_primary_term': 1} 
```

  可以看到index()方法会自动生成一个唯一id，当然我们也可以使用create()方法创建数据，不同的是create()需要手动指定一个id。

  修改数据也非常简单，我们同样需要指定数据的 id 和内容，调用 index() 方法即可，代码如下：

```python
data = {'content':"在北京学习python"}
#修改
result = es.index(index='article',body=data, id='GyJgb3MBuQaE6wYOApTh')
{'_index': 'article', '_type': '_doc', '_id': 'GyJgb3MBuQaE6wYOApTh', '_version': 2, 'result': 'updated', '_shards': {'total': 2, 'successful': 1, 'failed': 0}, '_seq_no': 6, '_primary_term': 1} 
```

  删除数据，可以调用 delete() 方法，指定需要删除的数据 id 即可

```python
#删除
result = es.delete(index='article',id='GyJgb3MBuQaE6wYOApTh')
print(result)
{'_index': 'article', '_type': '_doc', '_id': 'GyJgb3MBuQaE6wYOApTh', '_version': 3, 'result': 'deleted', '_shards': {'total': 2, 'successful': 1, 'failed': 0}, '_seq_no': 7, '_primary_term': 1} 
```

  查询数据，这里可以简单的查询全量数据：

```python
#查询
result = es.search(index='article')
print(result)

{'took': 1079, 'timed_out': False, '_shards': {'total': 1, 'successful': 1, 'skipped': 0, 'failed': 0}, 'hits': {'total': {'value': 5, 'relation': 'eq'}, 'max_score': 1.0, 'hits': [{'_index': 'article', '_type': 'blog', '_id': '1', '_score': 1.0, '_source': {'title': '我在北京学习人工智能', 'url': 'http://123.com', 'content': '在北京学习'}}, {'_index': 'article', '_type': 'blog', '_id': 'FyIdb3MBuQaE6wYO8JQR', '_score': 1.0, '_source': {'title': '你好', 'content': '你好123'}}, {'_index': 'article', '_type': 'blog', '_id': 'GCIeb3MBuQaE6wYOnpSv', '_score': 1.0, '_source': {'title': '你好', 'url': 'http://123.com', 'content': '你好123'}}, {'_index': 'article', '_type': 'blog', '_id': 'GSJfb3MBuQaE6wYOu5RD', '_score': 1.0, '_source': {'title': '你好', 'url': 'http://123.com', 'content': '你好123'}}, {'_index': 'article', '_type': 'blog', '_id': 'GiJfb3MBuQaE6wYO5pR4', '_score': 1.0, '_source': {'title': '你好', 'url': 'http://123.com', 'content': '你好123'}}]}}
```

  还可以进行全文检索，这才是体现 Elasticsearch 搜索引擎特性的地方。

```python
mapping = {
    'query': {
        'match': {
            'content': '学习 北京'
        }
    }
}

result = es.search(index='article',body=mapping)
print(result)

{'took': 4, 'timed_out': False, '_shards': {'total': 1, 'successful': 1, 'skipped': 0, 'failed': 0}, 'hits': {'total': {'value': 1, 'relation': 'eq'}, 'max_score': 4.075481, 'hits': [{'_index': 'article', '_type': 'blog', '_id': '1', '_score': 4.075481, '_source': {'title': '我在北京学习人工智能', 'url': 'http://123.com', 'content': '在北京学习'}}]}}
```

  可以看出，检索时会对对应的字段全文检索，结果还会按照检索关键词的相关性进行排序，这就是一个基本的搜索引擎雏形。

  除了这些最基本的操作，Elasticsearch还支持很多复杂的查询，可以参照最新的7.2版本文档：https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl.html