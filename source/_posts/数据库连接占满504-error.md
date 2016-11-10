---
title: 数据库连接占满504 error
date: 2016-10-25 10:12:02
tags: 数据库
categories: 技术相关
---

一般情况下数据库的插入，删除，更新操作会做 exception handle, 很少想到查询也需要这种操作。我在后台使用 UUID 作为数据库 ID 主键，当前端使用非规范的 uuid 进行查询的时候会触发 StatementError: (exceptions.ValueError) 这时候数据库链接并不会自己断开，导致多次非规范查询后数据库连接池被占满，返回 504 timeout 错误码。

所以不论什么时候都要保证一次请求过后相应的数据库 session 连接关闭。

<!--more-->

```py
def handle_exception(f):
    @wraps(f)
    def decoed(*args, **kwargs):
        raise_e = kwargs.pop("raise_e", True)
        try:
            return f(*args, **kwargs)
        except Exception as e:
            db_rollback()
            if raise_e is True:
                raise
            elif isinstance(raise_e, type) and issubclass(raise_e, Exception):
                if isinstance(e, raise_e):
                    raise
            db.session.close()          # 查询出错之后一定要保证数据库连接关闭，否则会占用连接导致504错误
    return decoed
```