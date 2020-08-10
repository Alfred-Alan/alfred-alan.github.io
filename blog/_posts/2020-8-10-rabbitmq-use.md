---
layout: post
title: '在Linux下安装rabbitmq并使用'
subtitle: '如何在Linux下安装rabbitmq并使用'
date: 2020-08-10
categories: 技术
tags: Ubuntu rabbitmq
image: /assets/img/blog/RabbitMQ-logo.png

---

如何在ubuntu下 安装rabbitmq 并使用

需进入root模式安装 或在所有命令前加``sudo``

由于rabbitMq需要erlang语言的支持，需要安装erlang：

```powershell
apt-get install erlang-nox # 安装 
安装之后输入 erl 查看是否成功 
```


添加公钥

```powershell
wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc | sudo apt-key add -
```


更新软件包

```powershell
apt-get update
```

安装 RabbitMQ

```powershell
apt-get install rabbitmq-server # 安装rabbitmq
```

查看 RabbitMq状态

```powershell
systemctl status rabbitmq-server # 如果有 Active: active (running) 说明处于运行状态
```

启动、停止、重启

```powershell
service rabbitmq-server start # 启动
service rabbitmq-server stop # 停止
service rabbitmq-server restart # 重启
```

执行了上面的步骤，rabbitMq已经安装成功。

如果要启用 web端可视化操作界面，我们还需要配置Management Plugin插件

```powersehll
rabbitmq-plugins enable rabbitmq_management # 启用插件
service rabbitmq-server restart # 重启
```

这里我新建了一个用户 方便使用 
查看用户

```powershell
rabbitmqctl list_users
```

添加管理用户

```powershell
rabbitmqctl add_user admin 123456 # 增加普通用户
rabbitmqctl set_user_tags admin administrator # 给普通用户分配管理员角色
```

用户创建完毕之后 就可以使用浏览器访问：http://服务器Ip:15672/ 来访问你的rabbitmq监控页面。



而后点击admin 发现刚刚创建的用户不允许访问主机

![20170520084716849](/assets/img/rabbitmq/20170520084716849.png)

<br>

点击用户名之后 设置一个默认的

![20170520084734185](/assets/img/rabbitmq/20170520084734185.png)

<br>

也可以用命令设置

命令设置权限方式为：

\>> rabbitmqctl set_permissions -p '/' yueer01 ".*" ".*" ".*"Setting permissions for user "yueer01" in vhost "/" ...

设置完成之后

![选择admin](/assets/img/rabbitmq/选择admin.png)

<br>

可以访问 

这里我使用Python来连接 rabbitmq

首先下载 pika 模块

```powershell
pip install  pika
```

创建发布者：

```python
# 创建凭证，使用rabbitmq用户密码登录
credentials = pika.PlainCredentials("admin", "123456")
cpara = pika.ConnectionParameters(host='0.0.0.0', port=5672, credentials=credentials)
connection = pika.BlockingConnection(cpara)


# 创建频道
channel = connection.channel()
# 新建一个queue1队列，用于接收消息
channel.queue_declare(queue='hello')

# 注意在rabbitmq中，消息想要发送给队列，必须经过交换(exchange)，
# 初学可以使用空字符串交换(exchange='')，
# 它允许我们精确的指定发送给哪个队列(routing_key=''),
# 参数body值发送的数据
channel.basic_publish(exchange='',
                      routing_key='hello',
                      body='hello world')
print("send sucessful")
# 程序退出前，确保刷新网络缓冲以及消息发送给rabbitmq，需要关闭本次连接
connection.close()
```

创建接受者：

```python
# 建立与rabbitmq的连接
credentials = pika.PlainCredentials("admin", "123456")
cpara = pika.ConnectionParameters(host='0.0.0.0', port=5672, credentials=credentials)
connection = pika.BlockingConnection(cpara)

# 创建频道
channel = connection.channel()
# 生命队列，消费者也要生命队列，因为我们不能确定消费者与生产者谁先启动。
channel.queue_declare(queue="hello")

# 回调函数，里面存放执行内容。
def callback(ch, method, properties, body):
    print(type(body))
    print(body.decode(encoding='utf-8'))


    # 有消息来临，callback，没有消息则夯住，等待消息
    channel.basic_consume(on_message_callback=callback, queue='hello', auto_ack=True)
    # 开始消费，接收消息
    channel.start_consuming()
```

此时编写的是普通轮训模式  当两个接受者开启之后 

第一条信息会优先发送给第一个的接受者，

第二条信息会发给第二个接受者

第三条轮回来 发送给第一个接受者



**取消自动应答**：如果消费者消费失败，消息会丢失，此时可以设置取消自动应答，

在 basic_consume中添加 auto_ack=False取消自动应答

并且在回调函数结尾处加入如下代码：ch.basic_ack(delivery_tag=method.delivery_tag)
手动应答，如果回调函数处理过程中发生错误，该条语句不会执行，消息回滚，等待下次消费。示例代码如下

```python
 def callback(ch, method, properties, body):
        print(type(body))
        # print(body.decode(encoding='utf-8'))
        print("消费者接收到了任务：%r" % body.decode(encoding='utf-8'))
        
        # 回调函数中进行手动应答。
        ch.basic_ack(delivery_tag=method.delivery_tag)


# auto_ack=False取消自动应答
channel.basic_consume(on_message_callback=callback, queue='queue1', auto_ack=False) 
```



**消息持久化** ： 如果消息进入消息队列后，RabbitMQ服务崩掉了怎么办，消息是不是丢失了，为了解决这个问题，可以设置持久化消息

在发布者和接受者新建队列时 设置消息持久化 

```python
# durable=True 设置消息持久化，即使rabbitmq重启也会保存消息，记得建立新的队列，因为以前的队列不支持持久化
channel.queue_declare(queue='queue1', durable=True)
```



## 交换机模式-发布/订阅模式

- 交换机的类型为**fanout**，此种模式下，每个消费者创建时，都会创建属于自己的队列，生产者会将消息传递到交换机，交换机传递到它绑定的所有队列上，此时所有生产者都会接收到消息。

#### 生产者

```python
import pika

# 创建凭证，使用rabbitmq用户密码登录
credentials = pika.PlainCredentials("admin", "123456")
cpara = pika.ConnectionParameters(host='0.0.0.0', port=5672, credentials=credentials)
connection = pika.BlockingConnection(cpara)

channel = connection.channel()
channel.exchange_declare(exchange='hello',
                         exchange_type='fanout', auto_delete=True)  # 发布订阅模式


channel.basic_publish(exchange='hello',
                      routing_key='',
                      body="Hello World!")
print("send sucessful")
connection.close()
```

#### 消费者

```python
import pika

# 建立与rabbitmq的连接
credentials = pika.PlainCredentials("admin", "123456")
cpara = pika.ConnectionParameters(host='0.0.0.0', port=5672, credentials=credentials)
connection = pika.BlockingConnection(cpara)

channel = connection.channel()

channel.exchange_declare(exchange='hello',
                         exchange_type='fanout', auto_delete=True)  # fanout发布订阅模式

result = channel.queue_declare("", exclusive=True)  # exclusive参数可以生成一个随机的队列名
queue_name = result.method.queue

channel.queue_bind(exchange='hello',
                   routing_key='',  # 发布订阅模式下，routing_key必须是空字符串
                   queue=queue_name)

def callback(ch, method, properties, body):
    print(" [x] %r" % body)

channel.basic_consume(queue=queue_name,
                      auto_ack=True,
                      on_message_callback=callback)

channel.start_consuming()
```

## 交换机模式-关键字模式

- 发布订阅模式下，所有消费者都会接收消息，如果生产者发送的消息只想让部分消费者接收，应该如何实现呢？此时可以考虑关键字模式，当交换机类型为**direct**时，为关键字模式

#### 生产者

```python
import pika

# 建立与rabbitmq的连接
credentials = pika.PlainCredentials("admin", "123456")
cpara = pika.ConnectionParameters(host='0.0.0.0', port=5672, credentials=credentials)
connection = pika.BlockingConnection(cpara)

channel = connection.channel()

channel.exchange_declare(exchange='hello',
                         exchange_type='direct', auto_delete=True)  # direct:关键字模式

message = "Hello World"
channel.basic_publish(exchange='hello',
                      routing_key='hello1',  # 设置关键字为error 
                      body=message)
print("send sucessful")
connection.close()
```

#### 消费者

```python
import pika

# 建立与rabbitmq的连接
credentials = pika.PlainCredentials("admin", "123456")
cpara = pika.ConnectionParameters(host='0.0.0.0', port=5672, credentials=credentials)
connection = pika.BlockingConnection(cpara)

channel = connection.channel()

channel.exchange_declare(exchange='hello',
                         exchange_type='direct', auto_delete=True)  # direct:关键字模式

result = channel.queue_declare("", exclusive=True)  # 自动生成队列名称
queue_name = result.method.queue

channel.queue_bind(exchange='hello',  
                   queue=queue_name,
                   routing_key='hello1'  # 绑定关键字error
                  )
def callback(ch, method, properties, body):
    print(" [x] %r" % body)

channel.basic_consume(queue=queue_name,
                      auto_ack=True,
                      on_message_callback=callback)

channel.start_consuming()
```

- 问题：如果消费者想绑定多个关键字，应如何实现？
  解答：消费者代码中多次调用channel.queue_bind方法，一次绑定一个关键字

```python
# consumer.py
channel.queue_bind(exchange='logs2',  
                   queue=queue_name,
                   routing_key='error'  # 绑定关键字error
                  )
channel.queue_bind(exchange='logs2',  
                   queue=queue_name,
                   routing_key='info'  # 绑定关键字info
                  )
channel.queue_bind(exchange='logs2',  
                   queue=queue_name,
                   routing_key='warning'  # 绑定关键字warning
                  )
```