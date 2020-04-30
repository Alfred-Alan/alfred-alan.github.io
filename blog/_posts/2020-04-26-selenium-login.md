---
layout: post
title: '使用selenium模块操作滑动验证及验证码'
subtitle: '如何使用selenium操作登录遇到的滑动模块'
date: 2020-04-26
categories: 技术
tags: selenium python
image: /assets/img/blog/selenium.jpg
---

#### 百度验证码识别

在日常登录中我们遇到图片验证码怎样识别呢

使用百度API接口来进行识别

到[https://login.bce.baidu.com/?account=](https://login.bce.baidu.com/?account=)注册百度账号

之后选择`人工智能-文字识别`

![选择类别](/assets/img/day5/选择类别.PNG)

选择创建

![选择创建](/assets/img/day5/选择创建.png)

![创建应用](/assets/img/day5/创建应用.png)

创建好打开应用后可以看到APIKey和Secret Key我们后面要用到

点击查看文档

![应用key](/assets/img/day5/应用key.png)

选择文字识别高精度版

![选择高精度版](/assets/img/day5/选择高精度版.png)

之后查看API文档 根据API内的说明来请求获取验证码

![使用文档](/assets/img/day5/使用文档.png)

这是我自己写好打包的函数

```python
def get_code(imgname):
    params = {
    	"grant_type": "client_credentials",
        "client_id": "bt9brCIZy4MYnHzBUqpHOYbf",
        "client_secret": "SimOcnn7D9BDnvsh0euXmMuncCrZeXXZ"}
    res = requests.get("https://aip.baidubce.com/oauth/2.0/token", params=params)
    token = res.json()['access_token']
    # 定义头部信息
    headers = {'Content-Type': 'application/x-www-from/urlencoded'}
    url = 'https://aip.baidubce.com/rest/2.0/ocr/v1/accurate_basic?access_token=' + token
    # 读取图片
    with open(imgname, 'rb')as f:
        tem_img = base64.b64encode(f.read())
    # 进行base64编码
    temp_data = {'image': tem_img}
    # 请求视图接口
    res = requests.post(url=url, data=temp_data, headers=headers)
    mycode = ''
    word_dict = res.json()['words_result']
    for key in word_dict:
        mycode += key['words']
    return mycode
```

#### 如何使用selenium来模拟登录及滑动验证

selenium是一款用来自动化测试的模块 

我们使用selenium 来进行模拟登录

![登录界面](/assets/img/day5/登录界面.png)

这是我自己写的一个界面

```python
from selenium import webdriver
from selenium.webdriver.common.action_chains import ActionChains
import requests
import time

browser = webdriver.Chrome()
def get_url():
    url = 'http://127.0.0.1:8080/login'
    # 请求url
    browser.get(url)
    # 获取input标签 并输入
    browser.find_element_by_xpath('//table/tr[1]/td[1]/input').send_keys('test3')
    time.sleep(2)
    browser.find_element_by_xpath('//table/tr[2]/td[1]/input').send_keys('a123456')
    time.sleep(2)
    # 获取验证码标签 并截图到本地
    browser.find_element_by_xpath('//img[@class="code"]').screenshot('1.png')
    # 调用打码函数 将图片名传进去 并填入验证码
    code = get_code('1.png')  //此接口调用上面的验证码函数
    print(code)
    browser.find_element_by_xpath('//table/tr[4]/td[1]/input').send_keys(code)
    # 获取滑动模块标签
    button=browser.find_element_by_xpath('//div[@class="dv_handler dv_handler_bg"]')
    # 声明动作实例
    action = ActionChains(browser)
    # 点击并且按住
    action.click_and_hold(button).perform()
    action.reset_actions()
    # 实际拖动像素和轨迹长度是有出入的
    action.move_by_offset(271, 0).perform()
    # 点击登录
    time.sleep(2)
    browser.find_element_by_xpath('//button[@class="h-btn h-btn-blue"]').click()

def main():
    get_url()


if __name__ == '__main__':
    main()
    time.sleep(5)
    browser.close()
```

