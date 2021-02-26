---
layout: post
title: '如何使用支付宝实现退款'
description: '如何使用支付宝的三方支付'
categories: [Python]
tags: [Pay]
image: /assets/img/blog/alipay.jpg

related_posts:
  - _posts/2020-06-16-alipay-use.md
  - _posts/2020-06-18-paypal-use.md
---
- Table of Contents
{:toc .large-only}

上次说到了如何使用支付宝搭建支付系统，而发现光支付是无法满足需求的。

你卖给用户商品了，用户发现并不好用，想退款怎么办？如果你不开发退款模块，那不就是老无赖一样了。

这就补全上篇博客的坑

## 支付宝退款

支付宝退款支持单笔交易分多次退款，多次退款需要提交原支付订单的商户订单号和设置不同的退款单号。一笔退款失败后重新提交，要采用原来的退款单号。总退款金额不能超过用户实际支付金额。

## 下载官方的sdk

```powershell
pip install alipay-sdk-python
```

## 实例化sdk对象

```python
# 直接将文本打开为数据流
app_private_key_string = open(os.path.join(BASE_DIR, "keys/app_private_2048.txt")).read()
alipay_public_key_string = open(os.path.join(BASE_DIR, "keys/alipay_public_2048.txt")).read()

alipay = alipay_sdk(
    appid="2016103000778082",
    app_notify_url=None,  # the default notify path
    app_private_key_string=app_private_key_string,
    # alipay public key, do not use your own public key!
    alipay_public_key_string=alipay_public_key_string,
    sign_type="RSA2",  # RSA or RSA2
    debug=True  # False by default
)
```

## 用这个对象来进行退款

```python
def refund(trade_id,money):
    order_string = alipay.api_alipay_trade_refund(
        # 订单号，一定要注意，这是支付成功后返回的唯一订单号
        out_trade_no=trade_id,
        # 退款金额，注意精确到分，不要超过订单支付总金额
        refund_amount=float(money),
        # 回调网址
        notify_url='http://127.0.0.1:8000/myapp/find_money/'
    )

    # 如果退款成功了
    if 'Success' in order_string:
        order = Order.objects.filter(trade_id=trade_id).first()
        order.pay_detail = 2
        order.save()
    return order_string
```

事实上，只要配合sdk实例化对象之后，在使用订单号进行退款是非常容易的事情

但是退款金额一定不要超过交易资格，只要在交易资格内还是可以分批次退款的