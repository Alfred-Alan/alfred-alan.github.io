---
layout: post
title: '使用Docker部署Gunicorn+Flask'
description: '如何使用docker来线上部署'
categories: [Docker]
image: /assets/img/blog/docker_flask.png
related_posts:
  - _posts/2020-04-27-docker-use.md
  - _posts/2020-7-15-docker-install-mysql.md
---
- Table of Contents
{:toc .large-only}

## docker

转载自https://v3u.cn/a_id_164

Docker其强大的跨平台特性，可以让我们在任何系统下部署项目，包括经常令人诟病的Windows，值得一提的是本次在Win10下部署项目的流程同样适用于Centos、Mac os、Ubuntu等系统，其兼容性可见一斑。

  关于Win10如何折腾和配置Docker，请参照这篇文章:[win10系统下把玩折腾DockerToolBox以及更换国内镜像源(各种神坑)](https://v3u.cn/a_id_149)

## 项目结构:

![img](/assets/img/docker_deploy/20200716100711_31478.png)

  manage.py是项目的入口文件，这里我们利用Sockert.io让Flask支持Websocket

```python
# file: 'manage.py'
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
import pymysql
from flask import request,jsonify
from flask_cors import CORS
from flask_socketio import SocketIO,send,emit,join_room, leave_room
import urllib.parse
import user_view

from celery import Celery
from datetime import timedelta

pymysql.install_as_MySQLdb()
 
app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "mysql://root:root@localhost:3306/md"
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = True

app.config['BROKER_URL'] = 'redis://localhost:6379'
app.config['CELERY_RESULT_BACKEND'] = 'redis://localhost:6379'
app.config['CELERY_ACCEPT_CONTENT'] = ['json', 'pickle']
app.config['REDIS_URL'] = 'redis://localhost:6379'
app.config['JSON_AS_ASCII'] = False

CORS(app,cors_allowed_origins="*")


app.register_blueprint(user_view.user)

db = SQLAlchemy(app)

socketio = SocketIO(app,cors_allowed_origins='*',async_mode="threading",message_queue=app.config['CELERY_RESULT_BACKEND'])


celery = Celery(app.name)
celery.conf.update(app.config)


celery.conf.CELERYBEAT_SCHEDULE = {
        
        "test":{
            "task":"get_cron",
            "schedule":timedelta(seconds=10)
        }

}




@celery.task(name="get_cron")
def get_cron():
    
    get_sendback.delay()
    

@celery.task()
def get_sendback():
    
    socketio.emit('sendback','message',broadcast=True)

@app.route('/task')
def start_background_task():
    get_sendback.delay()
    return '开始'
 

@app.route('/',methods=['GET','POST',"PUT","DELETE"])
def hello_world():
    #res = db.session.execute("insert into user (`username`) values ('123') ")

    # res = db.session.execute(" select id,username from user ").fetchall()

    # data = request.args.get("id")

    # #data = request.form.get("id")

    # print(data)

    # print(res)

    # #return 'Hello Flask'
    # return jsonify({'result': [dict(row) for row in res]})

    return jsonify({'message':'你好,Docker'})


@socketio.on('join')
def on_join(data):
    username = 'user1'
    room = 'room1'
    join_room(room)
    send(username + ' has entered the room.', room=room)

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
 
@app.route("/sendback",methods=['GET'])
def sendback():

    socketio.emit('sendback','message')

    return 'ok'
 
if __name__ == '__main__':

    socketio.run(app,debug=True,host="0.0.0.0",port=5000)
```
## 使用Gunicorn代理
  接下来使用Gunicorn+gevent来运行Flask项目，Gunicorn服务器作为wsgi app的容器，能够与各种Web框架兼容（flask，django等）,得益于gevent等技术，使用Gunicorn能够在基本不改变wsgi app代码的前提下，大幅度提高wsgi app的性能。那到底怎么提升性能？说简单点，Gunicorn 默认的网络模型是 select ，当我们把worker 替换成 gevent 后，则改为 epoll 监听模型，关于select、poll、epoll请参照这篇文章:[关于Tornado:真实的异步和虚假的异步](https://v3u.cn/a_id_107)，这里不再赘述。

  安装相应的库

```powershell
pip install gunicorn gevent --user
```

  编辑项目目录下的gunicorn.conf.py

```python
# file: 'gunicorn.conf.py'
workers = 3    # 进程数
worker_class = "gevent"   # 异步模式
bind = "0.0.0.0:5000"
```

  由于Gunicorn并不支持Windows环境，所以只需要写好配置，不需要运行。

  编辑项目目录下的requirements.txt文件，这里面都是我们项目所依赖的库

```powershell
flask==1.0.2
flask-cors
flask-socketio
flask-sqlalchemy
pymysql
celery
gunicorn
gevent
redis==3.3.11
```
## 编写Dockerfile
  随后在项目目录下创建一个 Dockerfile 文件，这个文件可以理解为打包镜像的脚本，你需要这个镜像做什么，就把任务写到脚本中，Docker通过执行这个脚本来打包镜像

```shell
# file: 'Dockerfile'
FROM python:3.6
WORKDIR /Project/myflask

COPY requirements.txt ./
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

COPY . .
ENV LANG C.UTF-8
CMD ["gunicorn", "manage:app", "-c", "./gunicorn.conf.py"]
```
## 打包镜像
  可以看到，我们项目的镜像首先基于python3.6这个基础镜像，然后声明项目目录在/Project/myflask中，拷贝依赖表，之后安装相应的依赖，这里在安装过程中我们指定了国内的源用来提高打包速度，最后利用gunicorn运行项目，值得一提的是，ENV LANG C.UTF-8是为了声明Docker内部环境中的编码，防止中文乱码问题。

  最后我们就可以愉快的打包整个项目了，在项目根目录下执行

```powershell
docker build -t 'myflask' .
```

 ![img](/assets/img/docker_deploy/20200716110704_84934.png)

  此时看到Docker通过读取Dockerfile文件来下载所需的基础镜像和依赖库，这里一定要指定Docker的下载源，否则速度会非常缓慢，打包好的镜像文件大概有1g左右。

## 查看项目镜像
  下载结束之后，可以看到myflask这个镜像已经静静躺在镜像库中了，运行

```powershell
docker images
```

![img](/assets/img/docker_deploy/20200716110709_50860.png)

## 启动容器
  然后我们就可以利用这个镜像来通过容器跑Flask项目了，运行命令

```powershell
docker run -it --rm -p 5000:5000 myflask
```

  这里的命令是通过端口映射把docker内部的端口5000映射到宿主机的5000端口上，后面的参数是镜像名称。我们看到，在Win10下，已经不可思议的通过Gunicorn把Flask跑起来了，这在之前没有Docker技术之前是不可想象的。

![img](/assets/img/docker_deploy/20200716110725_66656.png)

  通过网址访问一下，这里注意一点，就是Windows系统下，访问Docker容器需要通过分配的ip来访问，而不是我们常用的localhost。

![img](/assets/img/docker_deploy/20200716110756_47834.png)
## 测试
  完全没有任何问题。

![img](/assets/img/docker_deploy/20200716110737_84282.png)

  结语：到这里我们的 Docker+Flask + Gunicorn就部署完毕了，将这个镜像上传Dockerhub仓库，在任何时间、任何地点、任何系统上，只要连着网、只要我们想，就都可以在短短1分钟之内部署好我们的项目，这就是Docker技术对开发人员最好的馈赠