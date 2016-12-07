---
title: flask session 的坑
date: 2016-12-02 18:33:45
tags: 
		- flask
		- session
		- python
categories: 技术相关
---

最近将 token 授权改为 session-cookie 方式，后期搭建 https 配合，提高网站安全级别。
移动端的 API 请求采用 Authorization 中加入时限 token 的方式。

<!--more-->

# session cookie 简介

cookie 是浏览器存储在本地用于和服务器进行交互的凭证，由于 http 是无状态的，当我们需要持续与用户交互时，就有必要采用一种方式来产生一种持续的“会话”。而 cookie 就是实现这种会话的必备要素

当用户第一次向服务器发送（需要交互）的请求时，比如登陆操作，服务器的返回 response header 中会加入 set-cookie 字段，对应的值即为具有时效性的经过哈希的长文本。浏览器将这个值保存在本地，以后每次请求都将这个值加入 request header 中，key 名称即为 cookie。之后服务器程序会从 request header 中提取这个值，并判断是否过期或者无效。如果值有效且在有效期内，表明当前用户登录权限仍然有效，则继续进行其他逻辑处理。否则返回需要重新授权。

session 是服务器端用来保存用户信息的一种数据结构，session 可以借助很多方式来存储数据，比如文件系统，数据库等。flask 使用 python 字典来实现 session，数据存储在内存当中，核心功能就是判断session 是否有效，轻量简单。如果使用文件系统或者数据库实现 session 则还可以存储其他相关数据。


# flask session

flask 默认每次请求，都会在 response header 中，加入 set-cookie 字段，这样在设定的实效期内，只要用户向服务器发送请求，实效期就会向后推，导致授权一直有效。
而我们希望，当第一次授权后，用户正常使用，过了时效期用户需要重新授权。

通过设定 `SESSION_REFRESH_EACH_REQUEST = False ` 来将这种行为取消。同时需要将 session 变量的 permanent 属性设置为 True。 使用 `PERMANENT_SESSION_LIFETIME` 来设定时效期，默认为31天。

基本上通过设定这些变量就可以控制 seesion 行为。

遇到的坑就是在本地运行都是符合预期的，但上传到服务器就会出现错误，服务器返回 response 一直有 set-cookie header。折腾了半天，源码看了好几遍就是没发现原因，最后发现是 flask 版本不一致出的问题。心碎。。。