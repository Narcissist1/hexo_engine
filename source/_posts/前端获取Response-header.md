---
title: 前端获取Responsse header
date: 2017-02-17 15:20:55
tags:	response header
categories: 技术相关
---

由于最近有个需要前端获取 response header 的需求，代码完成后发现通过 getAllResponseHeaders() 或者 getResponseHeaders() 方法只能拿到 Content-type header. 
后来发现需要 response header 里面加上两个控制 header

Access-Control-Expose-Headers 和 Access-Control-Allow-Headers

在这两个 header 里面的 value 位置写入前端需要获取的 header 名就可以了。

