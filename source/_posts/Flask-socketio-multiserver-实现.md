---
title: Flask-socketio multiserver 实现
date: 2017-02-17 15:28:18
tags: websocket
categories: 技术相关
---

# Websocket

由于某些地方需要实时更新前端数据，想到使用 websocket, websocket 是基于 HTTP 协议的更高级通讯协议，可以实现客户端到服务器的实时交互会话。从而实现事件驱动交互而不需要客户端反复请求服务器。

<!--more-->

# Flask-socketio

主站使用了 Flask 框架，所以选择了 [Flask-socketio](https://flask-socketio.readthedocs.io/en/latest/) 库，除了提供 socket 通信还可以获取系统 app 配置，适合嵌入现有系统。客户端可以使用任何 [SocketIO](http://socket.io/) 提供的官方客户端，支持 Javascript, C++, Java 和 Swift 等语言。

# Multiserver

单 Server 的 websocket 比较简单，通过官方文档即可实现，这里只说一下多 server。多 server 文档上没有详细介绍。要实现多 server 需要使用消息队列，因为客户端会连接在各个 server 上，像消息广播或者分组消息发送就会受到影响，因为某个 server 收到的消息只会广播到连接在这个 server 上所有的客户端。使用消息队列后所有 server 都从确定的这个队列取消息发送，就可以避免有的客户端收不到消息的情况。

文档介绍 [Flask-socketio](https://flask-socketio.readthedocs.io/en/latest/) 实现多 server 可以使用 [redis](https://redis.io/) 或者 [rabbitmq](https://www.rabbitmq.com/) 消息队列。

我这里使用的是 [rabbitmq](https://www.rabbitmq.com/)。 

搭建好消息队列并启动服务后，新建 socketio 实例。

```py
app = Flask(__name__)
socket_io = SocketIO(app, async_mode=async_mode, engineio_logger=True, message_queue='amqp://username:password@localhost//',
                     channel='socket')
```

这里有个小问题，使用消息队列时，**app 必须要和 socket_io 在同一个文件里面新建**，如果分开创建之后通过 init_app() 方法初始化 soketio 实例，在多 server 时会报错。

这里 async_mode 我使用的是 gevent，虽然官方文档上说使用 eventlet 会效率更高一点，但我这里测试刚好相反，所以就使用了 gevent。





