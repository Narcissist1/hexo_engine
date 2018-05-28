---
title: Flask 源码备忘
date: 2018-04-10 18:25:11
tags: 
	- Flask
	- Local
	- LocalStack
categories: 技术相关
---

重读 Flask 源码，感觉比以前又学到了新的东西

# 全局变量

Flask 有一个设计的巧妙之处，就是对于全局变量的实现，比如 request, g, current_app, session 等等，用户在自己实现的代码的任何地方都可以通过
`from flask import ...` 来使用变量，而且保证了数据一致性和线程安全。flask 在底层使用了`werkzeug.local` 包的 `Local, LocalStack`，来保证线程安全。

<!--more-->
# Local, LocalStack

Local 源码
```py

class Local(object):
    __slots__ = ('__storage__', '__ident_func__')

    def __init__(self):
        object.__setattr__(self, '__storage__', {})
        object.__setattr__(self, '__ident_func__', get_ident)

    def __iter__(self):
        return iter(self.__storage__.items())

    def __call__(self, proxy):
        """Create a proxy for a name."""
        return LocalProxy(self, proxy)

    def __release_local__(self):
        self.__storage__.pop(self.__ident_func__(), None)

    def __getattr__(self, name):
        try:
            return self.__storage__[self.__ident_func__()][name]
        except KeyError:
            raise AttributeError(name)

    def __setattr__(self, name, value):
        ident = self.__ident_func__()
        storage = self.__storage__
        try:
            storage[ident][name] = value
        except KeyError:
            storage[ident] = {name: value}

    def __delattr__(self, name):
        try:
            del self.__storage__[self.__ident_func__()][name]
        except KeyError:
            raise AttributeError(name)
```

Local 相当于一个全局容器（虽然名字叫做Local）。他在内部维护一个全局字典(__storage__)，每个线程通过`get_ident` 函数获得当前线程唯一的id，在获取数据的时候（即通过执行`__getattr__`方法）通过这个id，来取得属于这个线程的数据，每个线程自己的数据结构也是字典。同样存储数据的时候也是如此。

LocalStack 源码

```py

class LocalStack(object):
    def __init__(self):
        self._local = Local()

    def __release_local__(self):
        self._local.__release_local__()

    def _get__ident_func__(self):
        return self._local.__ident_func__

    def _set__ident_func__(self, value):
        object.__setattr__(self._local, '__ident_func__', value)
    __ident_func__ = property(_get__ident_func__, _set__ident_func__)
    del _get__ident_func__, _set__ident_func__

    def __call__(self):
        def _lookup():
            rv = self.top
            if rv is None:
                raise RuntimeError('object unbound')
            return rv
        return LocalProxy(_lookup)

    def push(self, obj):
        """Pushes a new item to the stack"""
        rv = getattr(self._local, 'stack', None)
        if rv is None:
            self._local.stack = rv = []
        rv.append(obj)
        return rv

    def pop(self):
        """Removes the topmost item from the stack, will return the
        old value or `None` if the stack was already empty.
        """
        stack = getattr(self._local, 'stack', None)
        if stack is None:
            return None
        elif len(stack) == 1:
            release_local(self._local)
            return stack[-1]
        else:
            return stack.pop()

    @property
    def top(self):
        """The topmost item on the stack.  If the stack is empty,
        `None` is returned.
        """
        try:
            return self._local.stack[-1]
        except (AttributeError, IndexError):
            return None
```

LocalStack 是基于 Local 实现的一个栈数据结构，内部维护一个列表，这个列表存储在一个 Local 实例里面，所以每个线程其实都是维护自己的列表，因此也是线程安全的。

# App context

Flask 有两种上下文，其中之一就是 App 上下文，存储于 LocalStack 之中，之所以这样设计，是因为 Flask 允许创建多个 App, 而他们之间的上下文是相互独立的，所以需要 Stack 结构。

对于多个 App 的解释，在这个Stack Overflow 问题下的回答比较全面[链接](https://stackoverflow.com/questions/20036520/what-is-the-purpose-of-flasks-context-stacks)

Flask 著名的 g 变量就是存在于 App context 这一层, 另外我发现数据库ORM SQLAlchemy 通过 Python 代码生成的 SQL 语句也存在这个上下文里面。

# Request context

另外一个上下文就是 request context 了，顾名思义这个上下文存在于每次请求到来和结束之间，这个上下文中保存着和当前请求相关的一些信息，比如 request， session，flashes 等等。同样，request context 也保存在 LocalStack 里面，由于有些时候，你可能会希望使用 “内部转发” redirect 请求到其他地方处理，那么就会产生多于一个的 request context，这时候 Flask 通过将新产生的 context 压入栈中，最后压入的 context 在最上面，这样就保证了每次获取的都是当前正在处理的 context。

