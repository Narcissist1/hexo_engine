---
title: what is WSGI
date: 2016-04-05 12:17:27
tags: WSGI
categories: 技术相关
---

# Web Server Gateway Interface

Web Server Gateway Interface(WSGI) 是一个简单通用的接口规范，介于 web 服务器和 Python web 应用或者框架之间。最初由 Phillip J. Eby 在2003.12.7 提出，编于 PEP 333。之后被接受作为 Python
web 应用开发的标准。最近一个版本是 v1.0.1，也就是 PEP 3333，发布于 2010.12.26.

<!--more -->
## Idea

Python web 框架对于 Python 新手来说一直是一个问题，因为 web 框架的选择被 web 服务器的选择所限制。Python 应用程序通常被设计适配 CGI， FastCGI，mod_python等 web 服务器中的一种。

WSGI 作为一个底层接口被创建于 web 服务器和 web 应用程序或者框架之间，提升 web 应用程序开发的共性和可移植性。WSGI是基于现存的CGI标准而设计的。

## 规范概览

WSGI区分为两个部分：一为“服务器”或“网关”，另一为“应用程序”或“应用框架”。在处理一个WSGI请求时，服务器会为应用程序提供环境资讯及一个回呼函数（Callback Function）。当应用程序完成处理请求后，透过前述的回呼函数，将结果回传给服务器。

所谓的 WSGI 中间件同时实现了API的两方，因此可以在WSGI服务和WSGI应用之间起调解作用：从WSGI服务器的角度来说，中间件扮演应用程序，而从应用程序的角度来说，中间件扮演服务器。“中间件”组件可以执行以下功能：

* 重写环境变量后，根据目标URL，将请求消息路由到不同的应用对象。
* 允许在一个进程中同时运行多个应用程序或应用框架。
* 负载均衡和远程处理，通过在网络上转发请求和响应消息。
* 进行内容后处理，例如应用XSLT样式表。

## 示例程序

用Python语言写的一个符合WSGI的“Hello World”应用程序如下所示：

	def app(environ, start_response):
    	start_response('200 OK', [('Content-Type', 'text/plain')])
    	yield "Hello world!\n"
 
其中

* 第一行定义了一个名为app的callable[2]，接受两个参数，environ和start_response，environ是一个字典包含了CGI中的环境变量，start_response也是一个callable，接受两个必须的参数，status（HTTP状态）和response_headers（响应消息的头）。
* 第二行调用了start_response，状态指定为“200 OK”，消息头指定为内容类型是“text/plain”
* 第三行将响应消息的消息体返回。

[原文链接](https://en.wikipedia.org/wiki/Web_Server_Gateway_Interface)


