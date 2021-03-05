---
layout: post
title: '使用PayPal搭建跨境支付模块'
description: '如何使用PayPal做支付'
categories: [Python]
tags: [Pay]
image: /assets/img/blog/paypal.png

related_posts:
  - _posts/2020-06-16-alipay-use.md
  - _posts/2020-06-17-alipay-find-money.md


---
- Table of Contents
{:toc .large-only}

## PayPal

上篇提到了如何使用支付宝来搭建支付模块

忽然对支付模块有了兴趣

因为PayPal是面向全球的业务，如果公司业务是面向国际化的情况，那么PayPal是必不可少的支付系统。

PayPal支付的优势就是其业务网络遍布全球。目前PayPal的庞大网络覆盖了全球200多个国家，可提供20多种语言服务，并接受100多种货币付款和56种货币提现。同时，还允许在账户中持有25种货币余额。换句话说，只要付款人拥有一个PayPal账户，他就拥有了在200多个国家进行电子支付购物，并在需要服务的时候享受到母语支持的各种便捷服务

## 注册PayPal

首先注册官网 [PayPal](https://www.paypal.com) 

然后打开[开发者平台](https://developer.paypal.com/developer/accounts/) 选择create account 创建一个商家账号 和私人账号

<br/>

![创建账号](/assets/img/PayPal/create_account.png)

<br/>

创建之后 personal 就是用于支付的账户

然后选择修改下虚拟金钱方便测试

![修改价钱](/assets/img/PayPal/modify_price.png)

<br/>

email id 是用于付款的虚拟号码

而且密码还可以自定

![账户详情](/assets/img/PayPal/account_details.png)

<br/>

## 修改虚拟金额

![修改虚拟金额](/assets/img/PayPal/modify_virtual_amount.png)

<br/>

## 使用沙盒应用
跳转到[应用管理页面](https://developer.paypal.com/developer/applications/)

点击之后，他会默认生成一个沙盒应用

![Client ID](/assets/img/PayPal/Client ID.png)

<br/>

之后拿到 ```client id``` 和 ```secret``` 保存起来

## 使用paypal官方sdk

接下来就开始用paypal官方的 sdk来实现

```powershell
pip3 install paypalrestsdk
```

先实例化一下sdk对象

```python
# file: 'utils.py'
def get_paypal_object():
    paypalrestsdk.configure({
        "mode": "sandbox",  # sandbox代表沙盒
        "client_id": "",
        "client_secret": ""
    })
    return paypalrestsdk
```

创建一个视图类

```python
# file: 'views.py'
from myapp.utils import get_paypal_object

class paypal_view(APIView):
    def post(self,request):
        data = request.POST.dict()
        # 生成订单
        paypal=get_paypal_object()
        
        payment = paypal.Payment({
            "intent": "sale",
            "payer": {"payment_method": "paypal"},
            "redirect_urls": {
                "return_url": "http://127.0.0.1:8000/myapp/paypal/?snow_id="+data['query'],  # 支付成功跳转页面
                "cancel_url": "http://127.0.0.1:8000/myapp/paypal/?snow_id="+data['query'], # 取消支付页面
            },
            "transactions": [{
                "amount": {"total": data['order_price'], "currency": "USD"},
                "description": "在线教育平台购买"
            }]
        })
        # 创建订单
        if payment.create():
            for link in payment.links:
                if link.rel == "approval_url":
                    approval_url = str(link.href)
                    return redirect(approval_url)
        else:
            print(payment.error)
            return redirect(approval_url)
```

return_url 是支付成功后 回调的页面，把它回调进视图方法中。paypal会将一个支付者id回传，然后服务端需要验证支付才能真的完成支付，total是付款金额，精确到分，currency是币种，支持多钟类型的货币。

当Django的服务端创建好支付订单后，重定向到paypal的沙盒环境，这时候一定要使用沙盒的个人账号进行登录和支付。

支付之后会回调进指定的路由

```python
# ?paymentId=PAYID-L3SYORA3C031930S1733650J&token=EC-9TG269735K620131N&PayerID=ETYYRCDN8C3XJ  
paymentId 是支付订单id payer是支付者的id
```

然后根据这两个参数来判断是否支付

## 回调视图

```python
# file: 'views.py'
class paypal_view(APIView):
    def get(self,request):
        snow_id = request.GET.get('snow_id',None)
        paymentid = request.GET.get("paymentId",' ')  # 订单id
        payerid = request.GET.get("PayerID",' ')  # 支付者id

        order = Order.objects.using(db_connect_name).filter(snow_id=snow_id).first()
        info_url = 'http://127.0.0.1:8080/order_info?order_id='+snow_id

        paypal = get_paypal_object()
        payment = paypal.Payment.find(paymentid)
        if payment.execute({"payer_id": payerid}):
            # 成功支付
            order.sale_id = payment['transactions'][0]['related_resources'][0]['sale']['id']  # 流水号
            order.trade_time = payment['create_time'].replace("T",' ').replace("Z","")
            order.pay_detail = 1
            order.save()
            return redirect(info_url)
        else:
            print(payment.error)  # Error Hash
            return redirect(info_url)
```
## 注意

注意paypal与支付宝的流程不一样

他是支付之后，在验证的时候才会进行扣款，而不是点击支付直接付款

```python
payment.execute({"payer_id": payerid})
# 只有执行上句 才会支付成功 进行扣款
```
## 查询明细
候我们需要对交易流水进行一些核对，也可以通过接口查看交易明细

```python
payment = paypalrestsdk.Payment.find("订单号")
print(payment)
```

其返回的数据结构为

```python
{'id': 'PAYID-L3UXJWI36W563195B8405921',
      'intent': 'sale',
      'state': 'created',
      'cart': '3U13861672493404L',
      'payer': {
          'payment_method': 'paypal',
          'status': 'VERIFIED',
          'payer_info': {
                'email': 'sb-lkhld2313370@personal.example.com',
                 'first_name': 'John',
                 'last_name': 'Doe',
                 'payer_id': 'DFB7R27EAY5TG',
                 'shipping_address': {
                     'recipient_name': 'Doe John',
                     'line1': 'NO 1 Nan Jin Road',
                     'city': 'Shanghai',
                     'state': 'Shanghai',
                     'postal_code': '200000',
                     'country_code': 'C2'
                 },
                'country_code': 'C2'
          }
      },
      'transactions': [{
          'amount': {'total': '5.00', 'currency': 'USD'},
           'payee': {'merchant_id': 'NNB68GMNPQ6P4', 'email': 'sb-juj4v2315115@business.example.com'},
          'description': '这是一个订单测试',
          'item_list': {
              'shipping_address': {
                  'recipient_name': 'Doe John',
                  'line1': 'NO 1 Nan Jin Road',
                  'city': 'Shanghai',
                  'state': 'Shanghai',
                  'postal_code': '200000',
                  'country_code': 'C2'}
          },
          'related_resources': [{}]   # 如果支付执行了excute 这里就会有流水号
      }],
      'redirect_urls': {
          'return_url': 'http://localhost:8000/?paymentId=PAYID-L3UXJWI36W563195B8405921',
          'cancel_url': 'http://localhost:3000/paypal/cancel/'
      },
      'create_time': '2020-06-17T01:41:45Z',
      'update_time': '2020-06-17T02:10:11Z',
      'links': [
          {'href': 'https://api.sandbox.paypal.com/v1/payments/payment/PAYID-L3UXJWI36W563195B8405921', 'rel': 'self', 'method': 'GET'},
          {'href': 'https://api.sandbox.paypal.com/v1/payments/payment/PAYID-L3UXJWI36W563195B8405921/execute', 'rel': 'execute', 'method': 'POST'},
          {'href': 'https://www.sandbox.paypal.com/cgi-bin/webscr?cmd=_express-checkout&token=EC-3U13861672493404L', 'rel': 'approval_url', 'method': 'REDIRECT'}
      ]
    }
```

  
## 退款
如果用户想要退款的话，可以利用交易明细中的流水号进行退款业务。

```python
#退款
from paypalrestsdk import Sale

sale = Sale.find("流水号")
# Make Refund API call
# Set amount only if the refund is 
partialrefund = sale.refund({
    "amount": {
        "total": "5.00",
        "currency": "USD"}
})
# Check refund status
if refund.success():
    print("Refund[%s] Success" % (refund.id))
else:
    print("Unable to Refund")
    print(refund.error)
```

ok 这么一来就大功告成了

