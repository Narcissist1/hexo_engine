---
title: Python property decorator
date: 2016-04-12 14:12:23
tags: python
categories: 技术相关
---

# property() 方法

property() 方法返回了一个特殊的 [descriptor object](https://docs.python.org/2/howto/descriptor.html)， 它的目的就是创建一个类的属性，这个属性看起来用起来就像普通的属性一样，但是你可以提供自己的方法去控制它（设置它的值，获取它的值）。看起来就像一个操作定制化的属性。

操作这个属性的方法有三种 read, write, delete。当你创建一个属性时，你可以提供任意个或者全部的方法。

创建一个新式类 C，以及一个 p 属性：

	class C(...):
	    def R(self):
	        ...read method...
	    def W(self, value):
	        ...write method...
	    def D(self):
	        ...delete method...
	    p = property(R, W, D, doc)
	    ...

<!--more-->
* R 是一个不带参数的 `getter` 方法，返回属性的值，如果不提供此方法，任何读操作都会引起`AttributeError`
* W 是一个接受一个参数的 `setter` 方法，设置这个属性为这个参数的值，如果不提供此方法，任何写操作都会引起`AttributeError`
* D 是一个 `deleter` 方法，会删除这个属性，如果不提供此方法，删除操作会引起`AttributeError`
* doc 是一个文档字符串用来描述这个属性，调用方法
	
		C.p.__doc__

以下是一个小的类带有属性 x:

	class C(object):
	    def __init__(self):
	        self.__x=None
	    def getx(self):
	        print "+++ getx()"
	        return self.__x
	    def setx(self, v):
	        print "+++ setx({0})".format(v)
	        self.__x  =  v
	    def delx(self):
	        print "+++ delx()"
	        del self.__x
	    x=property(getx, setx, delx, "Me property 'x'.")

在 python 解释器中执行：

	>>> c=C()
	>>> print c.x
	+++ getx()
	None
	>>> print C.x.__doc__
	Me property 'x'.
	>>> c.x=15
	+++ setx(15)
	>>> c.x
	+++ getx()
	15
	>>> del c.x
	+++ delx()
	>>> c.x
	+++ getx()
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	  File "<stdin>", line 6, in getx
	AttributeError: 'C' object has no attribute '_C__x'

# property decorator
从 Python 2.6 开始这个方法支持装饰器用法 `@property` ，使用这个装饰器装饰一个函数，和用 `getter` 装饰它效果是一样的。另外这个装饰器本身又带有 `deleter` 和 `setter` 两个装饰器。可以使用被 `@property` 装饰过的函数去定义自己的 `setter` 和 `deleter` 方法。

比如你需要给你的类提供一个 state 属性，而你的 getter 方法返回一个私有的属性 ._state。你可以这样定义：

	@property
    def state(self):
        '''The internal state property.'''
        return self._state

这样 .state 将会是这个属性的 getter 方法，而且 文档字符串'''The internal state property.''' 也会被存储在属性中。

如果你需要 setter 和 deleter 方法：

	@state.setter
    def state(self, k):
        if not (0 <= k <= 2):
            raise ValueError("Must be 0 through 2 inclusive!")
        else:
            self._state = k
            
    @state.deleter
    def state(self):
        del self._state

    