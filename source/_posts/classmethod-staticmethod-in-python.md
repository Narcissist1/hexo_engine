---
title: 'classmethod & staticmethod in python'
date: 2016-04-25 15:55:26
tags: python
categories: 技术相关
---


# Difference between classmethod and staticmethod in python


```py
class A(object):
    def foo(self,x):
        print "executing foo(%s,%s)"%(self,x)

    @classmethod
    def class_foo(cls,x):
        print "executing class_foo(%s,%s)"%(cls,x)

    @staticmethod
    def static_foo(x):
        print "executing static_foo(%s)"%x    

a=A()
```

下面是调用 foo 方法的语句，a 实例将自己作为第一个参数传给 foo 方法：

	a.foo(1)
	# executing foo(<__main__.A object at 0xb7dbef0c>,1)

<!--more-->


---------------------------------------------------------

**调用类方法** 类将本身而不是创建的实例传给 class_foo 方法：

	a.class_foo(1)
	# executing class_foo(<class '__main__.A'>,1)

同样也可以通过类 A 直接调用 class_foo, 一般情况下，如果你定义了一个类方法，通常你会通过类而不是类创建的实例来调用它。

	A.class_foo(1)
	# executing class_foo(<class '__main__.A'>,1)

---------------------------------------------------------

**调用静态方法** 类本身或者类创建的实例都不会传给静态方法，静态方法和其他方法使用行为完全相同，但是你可以通过类或者类的实例来调用它。

	a.static_foo(1)
	# executing static_foo(1)

	A.static_foo('hi')
	# executing static_foo(hi)




