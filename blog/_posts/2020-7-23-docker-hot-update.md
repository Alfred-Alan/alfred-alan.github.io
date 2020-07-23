---
layout: post
title: '使用docker实现部署热更新'
subtitle: '如何使用docker实现部署热更新'
date: 2020-07-23
categories: 技术
tags: docker
image: /assets/img/blog/docker.png

---

我们在使用docker 部署 的时候

遇到了一个非常严谨的问题 

就是在项目更新的时候 总是需要重新build 

这样的话，浪费的大量的资源和时间，如何才能节省时间&资源进行优化处理 

而发现使用docker 热更新可以解决这一个问题 

其要点就是将本地的项目与容器内的项目进行共享 

这里我就使用flask来演示一下如何实现热更新 

<br>

首先将项目放到github中 

然后在服务器内使用git clone 将源项目克隆到服务器本地 

<br>

这里我使用gunicon 来代理flask的启动方式 

gunicorn可以结合gevent来进行部署，因此在高并发场景下也可适用，于是决定采用gunicorn进行部署。
gunicorn和supervisor会有一定的冲突，即使gunicorn中没有设置为后台启动，supervisor也只会管理gunicorn的master进程；
supervisor的重启服务对于无响应的Flask进程来说并不生效，不能很好地解决我的服务由于某些原因无法正常响应但又找不到方法解决的问题，因此暂时决定不用supervisor。

<br>

gunicorn 配置 如下 

```powershell
workers = 1    # 根据核心数 
worker_class = "gevent"   # 使用协程启动
bind = "0.0.0.0:5000"
reload = True  # debug 模式开启 

```

Dockerfile 配置如下

```powershell
FROM python:3.6
WORKDIR /Project/flask_project  # 容器内项目位置 

COPY requirements.txt ./ 
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

COPY . .
ENV LANG C.UTF-8
CMD ["gunicorn", "manage:app", "-c", "./gunicorn.conf.py"]

```

随后使用docker build 命令来生成 镜像包 

```powershell
docker build -t 'flask_project' .
```

查看 docker 镜像 

```powershell
[root@VM-0-12-centos flask_project]# docker images 
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
flask_project       latest              9c9a85db3f84        37 minutes ago      986 MB
myvue               latest              17537a1a2f41        23 hours ago        139 MB
docker.io/nginx     latest              0901fa9da894        12 days ago         132 MB
docker.io/mongo     latest              6d11486a97a7        2 weeks ago         388 MB
docker.io/python    3.6                 e0373ff33a19        3 weeks ago         914 MB
```

最后使用 docker -v 命令挂载执行 

```powershell
docker run -p 5000:5000 -v /root/project/flask_project:/Project/flask_project flask_project  
```

`/root/project/flask_project` 左边是服务器本机项目的位置 

``/Project/flask_project`` 右边是容器中项目的位置 

该方法就是 将服务器内的项目与 容器内的项目进行共享 

当项目发生更改 容器内也会更改 

![热更新前](/assets/img/dcoker_update/热更新前.png)

<br>

测试一下

```powershell
vim manage.py 
```

``` powershell
![热更新后](/home/bywlop/桌面/图片/热更新后.png)from app import *
import user_view
import get_order
import add_order
import nummber_register
app.register_blueprint(user_view.user)
app.register_blueprint(get_order.orders)
app.register_blueprint(add_order.order)
app.register_blueprint(nummber_register.number)

@app.route('/',methods=['GET','POST'])
def index():
    return '123456'  # 这里加一个6

# /get_file/61b7dbb7-6d69-4a0f-81c6-a1a2f05110c3.png
# 展示本地图片路由
@app.route('/get_file/<file_name>', methods=['GET'])
def get_file(file_name):
    try:
        response = make_response(
            send_from_directory('./static/', file_name, as_attachment=True))
        return response
    except Exception as e:
        return f"文件读取异常{e}"

if __name__ == '__main__':
    app.run(debug=True, host="0.0.0.0", port=5000)                                                         
```

![热更新后](/assets/img/dcoker_update/热更新后.png)

ok 完成