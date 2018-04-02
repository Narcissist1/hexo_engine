---
title: pyinstaller 备忘
date: 2018-04-02 10:07:03
tags: pyinstaller
categories: 技术相关
---

# 代码加密

[pyinstaller] 是将 Python 代码加密的一个方便的工具，当然也有些限制。

通过简单的命令即可将源代码编译成二进制文件。例：

	pyinstaller yourprogram.py

它通过分析文件的导入，会将所依赖的文件全部打包成一个独立的只依赖所运行的操作系统的文件夹，其中包含了可运行文件。

为了方便部署运行，可以将这个独立的文件夹基于其打包的操作系统制作成 docker image。

<!-- more -->

# 静态文件

[pyinstaller] 只会分析 py 文件并打包，对于项目使用的静态文件比如 html 和 pem 文件并不理会，所以需要在配置文件中单独指定，[pyinstaller] 的打包命令会生成一个 .spec 的配置文件， 其中的 datas 列表项，就是允许用户加入自己要使用的静态文件目录。
[文档](https://pyinstaller.readthedocs.io/en/stable/spec-files.html#adding-data-files)

# 其他依赖

由于 [pyinstaller] 通过 import 分析并导入包，但有些情况下需要在运行时导入包，这就会导致 import error, 所以 [pyinstaller] 提供了 hiddenimports 这个参数，通过手动指定进行导入。在使用 [Celery](http://www.celeryproject.org/) 时最多的使用了这个功能。

我的 hiddenimports 列表

```
hiddenimports=['celery.*', 'celery.loaders.app', 'celery.app.amqp', 'kombu.transport.redis', 'celery.backends', 'celery.apps', 'celery.events', 'celery.worker', 'celery.bin', 'celery.concurrency', 'celery.contrib', 'celery.fixups', 'celery.security', 'celery.task', 'celery.utils', 'celery.backends.redis', 'celery.app.events']

```

# 坑

项目中使用了 Flask-User 其中的底层依赖在使用 [pyinstaller] 打包时总是会有问题, 如果使用 pycryptodome 包就会报 ImportError: No module named 'Crypto'， 后来研究了一下换成了 pycrypto 就好了。


[pyinstaller]: https://www.pyinstaller.org/