---
layout: post
title: '如何构建支付宝第三方支付系统'
description: '如何使用支付宝的三方支付'
categories: [Python]
tags: [Pay]
image: /assets/img/blog/alipay.jpg

related_posts:
  - _posts/2020-06-17-alipay-find-money.md
  - _posts/2020-06-18-paypal-use.md
---
- Table of Contents
{:toc .large-only}

## 支付模块
在编写商城系统的时候，都必须要接触支付模块

在编写类似商城的网站的时候，都不可避免的接触到第三方支付功能。

国内做支付的公司有很多，其中阿里的支付宝也算是数一数二的了，这些年在互联网上关于支付宝的安全新闻又少之又少，说明了支付宝的安全系统还是很不错的。

今天来介绍使用如何使用支付宝第三方支付系统。

<br/>

## 准备工作

支付宝的验证系统是使用RSA算法和数字签名组成。

需要通过RSA算法生成一个私钥和一个公钥

它的验证流程是：

​		发方首先有一个公钥/私钥，它将要签名的报文作为一个单向散列函数的输入，产生一个定长的散列码，一般称为消息摘要。

　　使用发方的私钥对散列码进行加密生成签名。将报文和签名一同发出去。

　　收方用和发放一样的散列函数对报文运算生成一个散列码，同时用发放的公钥对签名进行解密。

　　如果收方计算得到的散列码和解密的签名一致，那么说明的确是发方对报文进行了签名而且报文在途中没有被篡改。

介绍一下window环境下如何生成 RSA秘钥

## 下载 openssl

首先需要到***[openssl官网](https://www.openssl.org/source/)***下载一个环境压缩包

![openssldownload](/assets/img/alipay/openssldownload.png)

解压之后输入```openssl```进入环境

```powershell
C:\Users\by wlop\Desktop\openssl-1.1.1g>openssl
OpenSSL>
```

## 使用命令生成公钥

```powershell
OpenSSL> genrsa -out rsa_private_key.pem   2048  #生成私钥
OpenSSL> rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem #生成公钥
OpenSSL> exit #退出OpenSSL程序
```

之后在同目录下就可以看见生成的两个秘钥

![秘钥位置](/assets/img/alipay/key_location.png)

这样就拿到了公钥和私钥

## 换取秘钥

还需要到[支付宝开发者中心](https://openhome.alipay.com/platform/developerIndex.htm)创建一个支付宝沙箱

![研发服务](/assets/img/alipay/R&D_service.png)

然后把生成的公钥(public_key)的内容复制，到RSA2秘钥中换取支付宝秘钥

注意不要带上-----BEGIN PUBLIC KEY----- 和-----END PUBLIC KEY-----![换取公钥](/assets/img/alipay/public_key.png)

<br/>

现在我们有了公钥，支付宝公钥，私钥，这三个秘钥

![位置](/assets/img/alipay/position.png)

把秘钥放进了项目中方便使用

<br/>

## 如何使用

在Django中集成支付接口的前置操作就是需要安装pycryptodome

pip3 install -i https://pypi.douban.com/simple pycryptodome

官方开发文档地址：https://docs.open.alipay.com/api

然后根据开发文档写一个基类

```python
# file: 'pay.py'
from datetime import datetime
from Crypto.PublicKey import RSA
from Crypto.Signature import PKCS1_v1_5
from Crypto.Hash import SHA256
from urllib.parse import quote_plus
from urllib.parse import urlparse, parse_qs
from base64 import decodebytes, encodebytes
import json
import requests

class AliPay(object):
    """
    支付宝支付接口(PC端支付接口)
    """

    def __init__(self, appid, app_notify_url, app_private_key_path,
                 alipay_public_key_path, return_url, debug=False):
        self.appid = appid
        self.app_notify_url = app_notify_url
        self.app_private_key_path = app_private_key_path
        self.app_private_key = None
        self.return_url = return_url
        with open(self.app_private_key_path) as fp:
            self.app_private_key = RSA.importKey(fp.read())
        self.alipay_public_key_path = alipay_public_key_path
        with open(self.alipay_public_key_path) as fp:
            self.alipay_public_key = RSA.importKey(fp.read())

        if debug is True:
            self.__gateway = "https://openapi.alipaydev.com/gateway.do"
        else:
            self.__gateway = "https://openapi.alipay.com/gateway.do"

    def direct_pay(self, subject, out_trade_no, total_amount, return_url=None, **kwargs):
        biz_content = {
            "subject": subject,
            "out_trade_no": out_trade_no,
            "total_amount": total_amount,
            "product_code": "FAST_INSTANT_TRADE_PAY",
            # "qr_pay_mode":4
        }

        biz_content.update(kwargs)
        data = self.build_body("alipay.trade.page.pay", biz_content, self.return_url)
        return self.sign_data(data)

    def build_body(self, method, biz_content, return_url=None):
        data = {
            "app_id": self.appid,
            "method": method,
            "charset": "utf-8",
            "sign_type": "RSA2",
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "version": "1.0",
            "biz_content": biz_content
        }

        if return_url is not None:
            data["notify_url"] = self.app_notify_url
            data["return_url"] = self.return_url

        return data

    def sign_data(self, data):
        data.pop("sign", None)
        # 排序后的字符串
        unsigned_items = self.ordered_data(data)
        unsigned_string = "&".join("{0}={1}".format(k, v) for k, v in unsigned_items)
        sign = self.sign(unsigned_string.encode("utf-8"))
        # ordered_items = self.ordered_data(data)
        quoted_string = "&".join("{0}={1}".format(k, quote_plus(v)) for k, v in unsigned_items)

        # 获得最终的订单信息字符串
        signed_string = quoted_string + "&sign=" + quote_plus(sign)
        return signed_string

    def ordered_data(self, data):
        complex_keys = []
        for key, value in data.items():
            if isinstance(value, dict):
                complex_keys.append(key)

        # 将字典类型的数据dump出来
        for key in complex_keys:
            data[key] = json.dumps(data[key], separators=(',', ':'))

        return sorted([(k, v) for k, v in data.items()])

    def sign(self, unsigned_string):
        # 开始计算签名
        key = self.app_private_key
        signer = PKCS1_v1_5.new(key)
        signature = signer.sign(SHA256.new(unsigned_string))
        # base64 编码，转换为unicode表示并移除回车
        sign = encodebytes(signature).decode("utf8").replace("\n", "")
        return sign

    def _verify(self, raw_content, signature):
        # 开始计算签名
        key = self.alipay_public_key
        signer = PKCS1_v1_5.new(key)
        digest = SHA256.new()
        digest.update(raw_content.encode("utf8"))
        if signer.verify(digest, decodebytes(signature.encode("utf8"))):
            return True
        return False

    def verify(self, data, signature):
        if "sign_type" in data:
            sign_type = data.pop("sign_type")
        # 排序后的字符串
        unsigned_items = self.ordered_data(data)
        message = "&".join(u"{}={}".format(k, v) for k, v in unsigned_items)
        return self._verify(message, signature)

    #请求退款接口
    def api_alipay_trade_refund(self,refund_amount,out_trade_no=None,trade_no=None,**kwargs):
        # 构造参数体
        biz_content = { "refund_amount":refund_amount}
        # 传递可选参数
        biz_content.update(**kwargs)
        # 判断使用站外订单还是支付宝订单
        if out_trade_no:
            biz_content["out_trade_no"] = out_trade_no
        if trade_no:
            biz_content["trade_no"] = trade_no
        # 构造支付接口地址
        data = self.build_body("alipay.trade.refund",biz_content)
        # 构造url
        url = self.__gateway+"?" + self.sign_data(data)
        # 请求接口
        r = requests.get(url)
        html = r.content.decode("utf-8")
        return html
```

## 在视同中使用

```python
# file: 'utils.py'
from mydjango.pay import AliPay

# 引入密钥
app_private_key_string = os.path.join(BASE_DIR, "keys/app_private_2048.txt")
alipay_public_key_string = os.path.join(BASE_DIR, "keys/alipay_public_2048.txt")

# 建立支付实例
def get_ali_object():
    app_id = "2016103000778082"  # APPID （沙箱应用）
    # 支付完成后，跳转的地址。
    return_url='http://127.0.0.1:8000/myapp/alipay/'
    alipay = AliPay(
        appid=app_id,
        app_notify_url=return_url,
        return_url=return_url,
        app_private_key_path=app_private_key_string,
        alipay_public_key_path=alipay_public_key_string,  # 支付宝的公钥，验证支付宝回传消息使用，不是你自己的公钥
        debug=True,  # 默认False,
    )
    return alipay
```

## 支付的视图

```python
# file: 'views.py'
class alipay_view(APIView):
    def post(self,request):
	    money = float(request.POST.get('money'))
        alipay = get_ali_object()
        # 生成支付的url
        query_params = alipay.direct_pay(
            subject="在线教育平台购买",  # 商品简单描述
            out_trade_no=订单id
            total_amount=money,  # 交易金额(单位: 元 保留俩位小数)
        )
        pay_url = "https://openapi.alipaydev.com/gateway.do?{0}".format(query_params)  # 支付宝网关地址
	    return redirect(pay_url)	
```

通过post方法请求到该函数内，使用支付宝基类传入参数构建一个用于支付的url

然后要创建函数接受支付后的返回

```python
# file: 'views.py'
class alipay_view(APIView):
    def get(self, request):
        alipay = get_ali_object()
        params = request.GET.dict()
# params={'charset': 'utf-8', 'out_trade_no': '4370097719021146113', 'method': 'alipay.trade.page.pay.return', 'total_amount': '200.00', 'trade_no': '2020061622001414240512024565', 'auth_app_id': '2016103000778082', 'version': '1.0', 'app_id': '2016103000778082', 'seller_id': '2088102181444124', 'timestamp': '2020-06-16 14:45:18'}
        sign = params.pop('sign', None)
        status = alipay.verify(params, sign)
        
        snow_id = params['out_trade_no']
        order = Order.objects.filter(snow_id=snow_id).first()
        info_url = 'http://127.0.0.1:8080/order_info?order_id='+snow_id
        # 使用回调参数会验证是否付款
        print(status)
        if status:  # 成功入库
            order.pay_detail = 1
            order.save()
            # 回跳指定路由
            return redirect(info_url)
        else:
            return redirect(info_url)
```

最后注意付款的账户要使用沙箱账号里的卖家账号

不可能有人会使用真实账号付款吧....



