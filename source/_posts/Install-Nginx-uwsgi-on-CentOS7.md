---
title: Install Nginx uwsgi on CentOS7
date: 2016-05-06 12:04:07
tags:
	- Nginx
	- uwsgi
	- python
categories: 技术相关
---

# Introduction

前些日子买了个 VPS，折腾着安装了 Nginx & uWSGI server, 写个备忘吧

# 安装组件

首先需要在 CentOS7 上安装必要的组件来运行程序，这里主要使用 yum 和 pip。

首先安装 EPEL

	sudo yum install epel-release

接下来是 Nginx web server 和 python 相关的库

	sudo yum install python-pip python-devel nginx gcc

安装 uWSGI server

	pip install uwsgi

<!--more-->

# 编写一个简单的 python app

这里我使用了 flask framework, pip 安装。

	pip install flask

简单的 demo 程序

```py

from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)


class HelloWorld(Resource):
    def get(self):
        return {'content': 'content change test'}


api.add_resource(HelloWorld, '/')

if __name__ == '__main__':
    app.run(debug=True)

```
编写一个 wsgi.py 组件来引入刚才我们编写的 app。

```py

from api import app

if __name__ == "__main__":
    app.run()

```

这样运行如下用命令，就可以在本地运行了：

	uwsgi --socket 0.0.0.0:8080 --protocol=http -w wsgi

使用 8080 端口， http 协议

# 配置 Nginx 和 uWSGI server

刚才运行的程序只是在本地，我们希望它可以服务外网，这就需要 Nginx 和 uWSGI 配合来做。下面是一些配置步骤。

## uWSGI config file

uWSGI server 可以通过配置文件启动，如果配置的项较长，显然通过文件启动会比较方便。
在项目根目录新建 myapp.ini 配置文件

	[uwsgi]
	module = wsgi:app

	master = true
	processes = 5
	uid = user

	socket = /run/uwsgi/myapp.sock
	chown-socket = user:nginx
	chmod-socket = 660
	vacuum = true

	die-on-term = true

配置文件指明了需要运行的 app, 使用 master mode 启动，使用5个 worker processes 来处理请求，使用 socket 来进行请求转发，效率更高更加安全。
vacuum 表明当 process 结束运行是自动删除 socket。

由于 uWSGI 和 systemd 对于 SIGTERM signal 的不同处理方式，我们需要最后一个 option 来让程序按照 systemd 期望的方式运行，uWSGI 会杀掉进程而不是 reload 它。

## systemd 配置

当系统重启时我们希望自动启动 uWSGI server, 这时候就需要用到 systemd，编写一个 systemd 配置文件。

	sudo nano /etc/systemd/system/uwsgi.service

文件内容：

	[Unit]
	Description=uWSGI instance to serve myapp
	[Service]
	ExecStartPre=-/usr/bin/bash -c 'mkdir -p /run/uwsgi; chown root:nginx /run/uwsgi'
	ExecStart=/usr/bin/bash -c 'cd /root/flasktest; uwsgi --ini flasktest.ini'

	[Install]
	WantedBy=multi-user.target

ExecStartPre 部分确保要有存放 socket 的目录，ExecStart 部分，首先切换到项目根目录，然后通过配置文件启动 uWSGI server。

[Install] section 指定了 enable 命令执行时需要的操作，基本上是指定哪些状态自动启动，我们这里指定当 multi-user mode 是启动。

保存配置文件后，接下来就可以通过 systemd 启动 uWSGI server 了。

	sudo systemctl start uwsgi

启动之后， 查看状态

	sudo systemctl status uwsgi

指定开机启动服务：

	sudo systemctl enable uwsgi

停止服务

	sudo systemctl stop uwsgi


## 配置 Nginx proxy server

在 /etc/nginx/conf.d/ 目录下新建 .conf 配置文件

	vim /etc/nginx/conf.d/flasktest.conf

内容如下

	server {
	    listen 8080;
	    server_name 104.224.160.89;

	    location / {
	        include uwsgi_params;
	        uwsgi_pass unix:/run/uwsgi/flasktest.sock;
	    }
	}

监听 8080 端口，IP地址作为 server name, 将 request pass 到我们指定的 socket 中。

测试 nginx 配置文件是否有问题

	sudo nginx -t

使用 systemd 开启 nginx 服务

	sudo systemctl start nginx

开机自启动

	sudo systemctl enable nginx

# 总结

Nginx 作为静态服务器效率极高，近年来被广泛推广。配合 uWSGI 使用 systemd 或者 supervisor 作为运维工具，维护 Nginx 和 uWSGI 进程，记录 log, 监控服务等等，由于 uWSGI 支持大部分上层框架，web framework 可以方便的根据自己的业务逻辑进行选择。

