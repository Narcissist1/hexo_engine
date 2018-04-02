---
title: WeChat Pay SDK
date: 2017-12-21 17:21:22
tags: WeChatPay
categories: 技术相关
---

最近公司业务需要接入微信支付，发现官方并没有提供相应的 Python 版本 SDK，去 GitHub 上找了一下发现也没有合适的，索性自己写了一个打成包发布了，这样以后有相应需求只需要 `pip install` 一下就好了。

[Code](https://github.com/Narcissist1/wechat-pay)
[Doc](https://wechat-pay-sdk.readthedocs.io/en/latest/)
[WeChat Official API](https://pay.weixin.qq.com/wiki/doc/api/index.html)

<!-- more -->

# WeChatPay

核心逻辑处理类，包括了数据准备，签名生成，验证签名等基本方法。微信支付基本API(统一下单，订单查询，验证支付提醒等等)都在这个类中实现。

# WeChatResult

微信返回结果类，将微信的返回做了一些基本处理，首先将 xml 格式的字符串转换成 Python 字典， 然后通过判断 return_code, result_code 等字段，判断此次的业务逻辑请求是否成功。如果成功，将其中的一些核心字段提取出来，保存为类的属性(个人比较喜欢通过类属性访问数据，看起来比字典简洁一些)。如果失败，将相应的错误信息提取保存。

# Other things

某些API的调用需要附带证书，微信官网提供了四个分别是（apiclient_cert.p12，rootca.pem， apiclient_cert.pem， apiclient_key.pem），如果你用的是 [requests](http://docs.python-requests.org/en/master/) 发起的请求你需要的是 apiclient_cert.pem， apiclient_key.pem 两个证书，将他们保存在本地的某个路径，并在初始化 WeChatPay 的时候传入路径即可。

微信的支付成功通知，退款通知等，并不保证会发到你的服务器，而且还有可能多发。所以要写好相应的处理逻辑，避免多次修改订单数据。







