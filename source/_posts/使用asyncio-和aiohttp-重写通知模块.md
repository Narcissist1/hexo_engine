---
title: 使用asyncio 和aiohttp 重写通知模块
date: 2018-05-25 18:09:12
tags: 
    - asyncio
    - python
    - aiohttp
categories: 技术相关
---

作为 python 3 最为重要的更新之一，asyncio 包使得使用 Python 编写IO异步程序变得更加简单。[aiohttp] 则在底层支持了这种异步操作。

# asyncio

协程的本质在于通过类似生成器的概念，在程序某一点将控制权交出，同时可以产生数据（也可以不产生），主程序获得控制权之后执行一点，再次激活协程（控制权交回）直到，协程执行完毕（抛出StopIteration异常）。主程序可以同时开启多个协程，也可以将协程串联起来，即协程之中再次调用协程（yield from），程序执行的控制权在主程序和协程之间来回切换从而实现并发。

<!--more-->

而程序速度提高的原因在于IO操作的耗时远高于CPU计算，当协程遇到IO操作的时候会把控制权交出，这时候就允许其他协程做其他的事情，而不是阻塞在那里。

在早期版本中编写协程需要使用装饰器 @asyncio.coroutine, 而在协程中调用其他协程需要使用 yield from

3.5 版本中新加了 async 和 await 关键字，使得编写协程的句法更加简洁。

# aiohttp

[aiohttp] 包含了客户端和服务端的异步库，可以方便的编写非阻塞程序。

在服务端，[aiohttp] 提供了一个最基本的 web 框架，可以像下面这样快速的创建一个app

```py
from aiohttp import web

def create_app():
    app = web.Application()
    return app
```

虽然还无法和 Django，Flask 这些框架相提并论，但 [aiohttp] 提供了编写一个 web 服务最基本的支持。

# 通知模块

研究了一下 asyncio 模块，就想着做点实践，正好通知部分涉及的 API 比较少，而且消息数据存储在 MongoDB 上，在逻辑上比较好分离出来单独做成一个服务。

由于一个完整的 asyncio 应用，需要底层IO模块支持协程式交互，所以我采用了 [motorengine] 库作为我的数据库 ORM。

## 定义数据模型

使用 [motorengine] 定义数据模型和使用 [MongoEngine](http://docs.mongoengine.org/) 很像。

下面是一个例子：

```py
from datetime import datetime
from uuid import uuid4
from motorengine.document import Document
from motorengine.fields import StringField, DateTimeField, BooleanField

class AccountMessage(Document):
    nid = StringField(required=False, default=str(uuid4()), max_length=40)
    user_id = StringField(required=False, default='', max_length=40)
    username = StringField(required=False, default='', max_length=50)
    message_type = StringField(required=True, default='', max_length=50)
    team_name = StringField(required=False, default='', max_length=100)
    reader_id = StringField(required=False, default='', max_length=40)
    create_time = DateTimeField(default=datetime.now)
    read = BooleanField(default=False)
```

## 查询数据库

通过 [motorengine] 查询数据库的方法也很简单，不过由于使用了非阻塞的方法，需要使用 await 关键字获取数据，同时当前方法也应该用 async 关键字将其标记为协程方法。而调用该方法的其他方法也应该使用 await。

```py
async def get_team_message(data, user_id):
    team_id = data.get('team_id', None)
    per_page = int(data.get('pageSize', 12))
    page_num = int(data.get('pageNo', 1))
    _self = data.get('self', 'false')

    q = TeamMessage.objects.filter(team_id=team_id, reader_id=user_id)
    if _self == 'true':
        q = q.filter(Q(user_id=user_id) | Q(receiver_id=user_id))
    count = await copy(q).filter(read=False, receiver_id=user_id).count()

    skips = page_num * (page_num - 1)
    q = q.order_by('create_time', direction=DESCENDING).order_by(
        'read', direction=ASCENDING).skip(skips).limit(per_page)
    rv = await q.find_all()
    count = await copy(q).filter(read=False).count()
    total = await copy(q).count()
    return rv, total, count

data, total, count = await get_team_message(data, uid)
```

有一点要注意的是，通过 filter 返回的 query 只能使用一次，因此相同的 query 语句多次查询数据库需要使用 copy 来生成新的实例。

[aiohttp]: https://aiohttp.readthedocs.io/en/stable/
[motorengine]: https://motorengine.readthedocs.io/en/latest/
