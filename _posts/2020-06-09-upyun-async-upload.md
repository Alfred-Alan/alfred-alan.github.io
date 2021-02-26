---
layout: post
title: 'django结合又拍云实现异步上传'
description: '如何使用异步队列来上传文件'
categories: [Python]
tags: [Django,Celery]
image: /assets/img/blog/upyun.jpg

---

之前在写项目的时候想到这么一个场景：

是基于视频播放的这么一个前端，而当有用户上传视频的时候，从前端向后端发送数据。

当在视图函数中上传到云端的话，如果视频过大 那么就会一直上传 而过长时间无法返回。

因为是前后端分离，相当于等待了两次上传时间， 前端把视频传输到后端，后端把视频上传到云储存成功后才能返回。

我们要做的是 前端传输过来视频之后 把文件流交给异步任务来上传，视图直接返回完成。



```python
def post(self,request):
    title = request.POST.get('title')
    file = request.FILES.get('file')
    size = int(request.POST.get('size'))
    print(title)
    # 如果判断文件大小
    if size > 1024 * 1024 * 1024:  # 大于1G
        cut_size = 1024*1024*40
    if size > 1024 * 1024 * 100:  # 大于100M
        cut_size = 1024*1024*20
    elif size > 1024 * 1024 * 50:  # 大于50M
         cut_size = 1024*1024*10
    else:  # 小于50M
         cut_size = 1024*1024*2
    # chunks(chunk_size=按大小分块)将文件流分块放进列表
    # 因为异步函数不能接受二进制 所以要把整个列表转换为str
    file_list=str([i for i in file.chunks(chunk_size=cut_size)])
    # 将文件流传输给异步任务
    res=tasks.upload_file.delay(title,file_list,file.name,cut_size)

    return Response({'code':200,'msg':'文件ok','second':len(file_list)})
```

异步任务tasks代码

这里的想法是 将文件流分割 再使用多线程并发上传

```python
import upyun
import threading

@app.task
def upload_file(title,file_list,name,cut_size):
	# 实例化又拍云对象
	up = upyun.UpYun('服务名', '操作者', '操作者密码')
    # up.init_multi_uploader(文件名，part_size=上传的大小) 声明并发上传对象
	uploader = up.init_multi_uploader('/%s/%s'%(title,name),part_size=cut_size)
	thread_list=[]
	# eval在将相似列表的str 强转成真正的列表
	for i,v in enumerate(eval(file_list)):
		print(i)
		# 多线程分块上传
		t = threading.Thread(target=uploader.upload, args=(i,v))
		thread_list.append(t)
     # 循环启动线程
	for t in thread_list:
		t.start()
		t.join() # 并且阻塞等待
        
	# 声明完成上传
	res = uploader.complete()
	return '成功'
```

这样就解决的真正的异步上传

用户将视频上传到后端之后  后端把文件流叫给异步任务来上传  告诉用户已完成



