---
layout: post
title: 'Flask结合Socket.io+vue2.0实现在线客服系统'
description: 'Flask结合Socket.io实现在线客服系统'
categories: Python
tags: [Socket,Flask]
image: /assets/img/blog/socketio.png

---
- Table of Contents
{:toc .large-only}

## Soket.IO

参考链接   https://v3u.cn/a_id_158
Soket.IO 就是一个封装了 Websocket、基于 Node 的 JavaScript 框架，包含 client 的 JavaScript 和 server 的 Node（现在也支持python,go lang等语言）。其屏蔽了所有底层细节，让顶层调用非常简单，另外，Socket.IO 还有一个非常重要的好处。其不仅支持 WebSocket，还支持许多种轮询机制以及其他实时通信方式，并封装了通用的接口。这些方式包含 Adobe Flash Socket、Ajax 长轮询、Ajax multipart streaming 、持久 Iframe、JSONP 轮询等。换句话说，当 Socket.IO 检测到当前环境不支持 WebSocket 时，能够自动地选择最佳的方式来实现网络的实时通信，这一点就比websocket要智能不少。

## 下载需要的包

```powershell
pip install flask
pip install flask-cors
pip install flask-socketio
```

  分别安装Flask本地，跨域模块，以及socketio模块

  适当升级你的pip，注意版本不要过低，下面是本次demo的版本号

```powershell
Flask                 1.1.1
Flask-Cors            3.0.8
Flask-SocketIO        4.3.0
Flask-SQLAlchemy      2.4.1
```

## 创建flask
随后我们简单写一个flask的入口启动文件 manage.py

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
import pymysql
from flask import request,jsonify
from flask_cors import CORS
from flask_socketio import SocketIO,send,emit
import urllib.parse

pymysql.install_as_MySQLdb()
 
app = Flask(__name__)

CORS(app,cors_allowed_origins="*")

socketio = SocketIO(app,cors_allowed_origins='*')
 
@socketio.on('message')
def handle_message(message):
    message = urllib.parse.unquote(message)
    print(message)
    send(message,broadcast=True)

@socketio.on('connect', namespace='/chat')
def test_connect():
    emit('my response', {'data': 'Connected'})

@socketio.on('disconnect', namespace='/chat')
def test_disconnect():
    print('Client disconnected')
 
if __name__ == '__main__':
    socketio.run(app,debug=True,host="0.0.0.0",port=5000)
```
## 注意
  这里简单说一下需要注意的地方，实例化socketio对象的时候，要加上cors_allowed_origins来设置跨域，前后端分离项目让人伤脑筋的地方就是浏览器同源策略问题，而跨域最好由server端来单独配置，这样的好处是当多个前端项目同时共用一套微服务接口时，就不用每个前端项目都配置一次跨域了。

  我们写了三个基于socketio的视图方法，connect和disconnect顾名思义，当clinet发起连接或者断开时我们可以及时捕获到，而message方法就是前后端进行消息通信的重要方法。



  发送消息的时候我们加了一个broadcast参数，这是socket.io极具特色的功能，类似广播的效果，可以同时给不同链接的client发送消息，即可以用于聊天，也可以用来做消息推送。

  最后需要注意的一点是，client发送消息时，最好用urlencode编码一下，这样可以解决中文乱码问题，而在server端，我们用urllib.parse.unquote()来进行解码操作。

## 启动后端服务

```powershell
python3 manage.py
```

![img](/assets/img/socket.io/20200624210612_96391.png)

  服务正常启动在5000端口上，就说明后端没有问题了。

  随后我们来配置前端(client)，前端采用vue2.0框架来驱动，也需要安装socket.io模块

## vue使用socket.io
```powershell
npm install vue-socket.io@2.1.0
```

  这里一定要指定版本号来安装，版本是2.1.0，因为该依赖的最新版在vue2.0项目中编译时会报错

  在入口文件main.js中引用

```js
import VueSocketio from 'vue-socket.io';

Vue.use(VueSocketio,'http://127.0.0.1:5000');
```

  注意链接的url是后端服务的地址+端口，千万不要加其他url后缀或者命名空间

  新建一个index.vue组件来进行模拟用户链接

```html
<template>
  <div>
	<div v-for="item in log_list">
	{{item}}
  </div>
	<input v-model="msg" />
	<button @click="send">发送消息</button>
</div>
</template>

<script>

export default {
  data () {
    return {
	  msg: "",
	  log_list:[]
    }
  },
  //注册组件标签
  components:{

  },
  sockets:{
    connect: function(){
      console.log('socket 连接成功')
    },
    message: function(val){
	  console.log('返回:'+val);
	this.log_list.push(val);
	}
},
  mounted:function(){
},
  methods:{

	send(){
	  this.$socket.emit('message',encodeURI("用户:"+this.msg));
    },

  }
}

</script>
 
<style>



</style>
```

## 启动vue服务

`npm run dev`

  访问前端页面 http://localhost:8080 可以看到服务成功链接

![img](/assets/img/socket.io/20200624220630_72542.png)

  这时我们可以尝试再做一个后台客服的组件页面item.vue，模拟用户和客服分别在不同的电脑进行聊天的场景

```html
<template>
  <div>
	<div v-for="item in log_list">
	{{item}}
  </div>
	<input v-model="msg" />
	<button @click="send">发送消息</button>
</div>
</template>
 
<script>


export default {
  data () {
    return {
	  msg: "",
	  log_list:[]
    }
  },
  //注册组件标签
  components:{

  },
  sockets:{
    connect: function(){
      console.log('socket 连接成功')
    },
    message: function(val){
	  console.log('返回:'+val);
	  this.log_list.push(val);
	}
},
  mounted:function(){
  	
},
  methods:{

	send(){
	  this.$socket.emit('message',encodeURI("客服:"+this.msg));
    },

  }
}

</script>
<style>
</style>
```



## 效果

![img](/assets/img/socket.io/20200624220655_38621.png)

  整个流程还是相对简单的，比起django的dwebsocket模块，socket.io显然更加灵活和方便，如果需要做一些主动推送任务，也可以利用socket.io的广播功能，其原理和实时聊天是一样的。
