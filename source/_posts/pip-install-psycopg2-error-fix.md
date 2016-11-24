---
title: pip install psycopg2 error fix(Mac)
date: 2016-11-21 22:28:35
tags: python
categories: 技术相关
---

最近将开发环境迁移到 virtualenvwrapper 上，需要重新安装项目依赖。由于后台数据库使用的是 Postgresql，需要安装响应的 python 驱动模块 psycopg2，安装时系统反馈了如下错误

```
ld: library not found for -lssl
clang: error: linker command failed with exit code 1 (use -v to see invocation)
'cc' failed with exit status 1
```

Google 了好久发现是升级 Xcode 的时候系统自动删除了 command line tools。

**解决办法**

`xcode-select --install`

之后再运行

`pip install psycopg2` 就不会报错了