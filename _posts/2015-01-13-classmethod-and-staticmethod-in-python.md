---
layout: post
title: "@classmethod and @staticmethod in Python"
description: ""
category: [Python, OOD]
---

## Why

[Python中的类型设计][1] 这篇文章里面介绍了 Python 在面向对象设计的时候，对于一个类型来说，其方法分为下面三种：

- __类实例方法__：最常见的类实例的方法，非私有的类实例方法只能通过类实例进行调用，并在调用的时候隐式传递类实例作为第一个参数，该参数的形参名可以为任何合法变量名，但一般使用 `self`。
- __类方法__：使用 `@classmethod` 修饰器修饰的类型的方法，非私有的类方法可以通过类型本身或者类实例进行调用，并在调用时隐式传递类型本身作为第一个参数，该参数的形参名可以为任何合法变量名，但一般使用 `cls`。
- __类静态方法__：使用 `@staticmethod` 修饰器修饰的类的静态方法，非私有的类静态方法也可以通过类型本身或者类实例进行调用，但是没有任何附加参数被隐式传递。方法内部调用 __类属性__ 或者 __类方法__ 的时候需要使用类型全名进行调用。

其中，__类实例方法__ 比较容易理解，和 C++/C#/JAVA 等面向对象编程语言中的类实例方法基本相同（Python 中基类的方法都可以在派生类中重写）。而 __类方法__ 和 __类静态方法__ 比较容易混淆。我们在进行类型设计的时候需要了解它们的适用场景并合理设计。

## What

本文整理总结一下 __类方法__ 和 __类静态方法__ 的典型应用场景和区别。

## 正文

__类方法__ 和 __类静态方法__ 的 __相同点__：

- 非私有 __类方法__ 和 __类静态方法__ 都可以通过类实例和类本身进行调用。
- 都可以通过继承被派生类拥有并重写。
- 都无法在方法内部调用 __类实例属性__ 和 __类实例方法__。

__类方法__ 和 __类静态方法__ 的 __不同点__：

- __类方法__ 在调用时会隐式传递类型作为第一个参数；而 __类静态方法__ 不会隐式传递任何值。
- __类方法__ 的方法体中可以通过传递进来的 `类型实参` 调用该类型的 `类属性` 和 `类方法`；而 __类静态方法__ 内部不知道任何关于类型或者类实例的信息，但是可以通过类型本身的类型全名调用 `类属性` 和 `类方法`。

二者之间的不同点决定了二者不同的适用场景：

- 由于 _类方法__ 在调用时会隐式传递类型作为第一个参数，而且可以通过类型实参调用该类型的 __类方法__，这就让我们可以在基类中定义获取类实例的 __factory 类方法__，之后在派生类中调用的时候生成派生类的实例。这种情况下如果使用 __类静态方法__，那由于必须在方法体中显示指定实例化的基类类型，其被派生类继承后生成的任然是基类的实例，不是我们期望的了。
- 对于和类型相关，但是可能和外部函数同名，而且我们希望其能被派生类重用时，就可以使用 __类静态方法__。

我们看一个实例：

{% highlight python %}
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Created by xiaohui96@gmail.com on 2014/12/25

__author__ = 'xiaohui96@gmail.com'

"""
类方法和类静态方法的实例
"""


class Base(object):

    # 类属性
    ClassProperty = 0

    def __init__(self, first, second):
        # 类实例属性
        self.InstanceProperty = ""
        self.InstanceProperty1 = first
        self.instanceProperty2 = second

    def instance_method(self, msg):
        self.InstanceProperty = msg
        print self.__class__.__name__ + " instance_method" + self.InstanceProperty

    @classmethod
    def class_method(cls, msg):
        print cls.__name__ + " class_method" + msg

    @staticmethod
    def static_method(msg):
        print "static_method" + msg

    @classmethod
    def instance_factory_classmethod(cls, first):
        return cls(first, 100)

    @staticmethod
    def instance_factory_staticmethod(frist):
        return Base(frist, 200)


class DerivedA(Base):

    ClassProperty = ""


class DerivedB(Base):

    @classmethod
    def class_method(cls, msg):
        print cls.__name__ + " class_method" + msg

    @staticmethod
    def static_method(msg):
        print "Overrided static_method" + msg


if __name__ == "__main__":
    DerivedA(1, 2).instance_method(" called by instance")
    DerivedA(1, 2).class_method(" called by instance.")
    DerivedA.class_method(" called by Derived class itself.")
    DerivedA(1, 2).static_method(" called by instance.")
    DerivedA.static_method(" called by Derived class itself.")
    DerivedA.ClassProperty = "DerivedA.ClassProperty"
    print DerivedA.ClassProperty

    DerivedB(1, 2).class_method(" called by instance.")
    DerivedB(1, 2).class_method(" called by instance.")
    DerivedB.class_method(" called by Derived class itself.")
    DerivedB(1, 2).static_method(" called by instance.")
    DerivedB.static_method(" called by Derived class itself.")
    print DerivedB.ClassProperty

    # 通过类静态方法获取类实例
    print isinstance(DerivedA.instance_factory_staticmethod(1), DerivedA)
    # 通过类方法后去类实例
    print isinstance(DerivedB.instance_factory_classmethod(1), DerivedB)
{% endhighlight %}

运行后会看到下面的输出：

{% highlight text %}
DerivedA instance_method called by instance
DerivedA class_method called by instance.
DerivedA class_method called by Derived class itself.
static_method called by instance.
static_method called by Derived class itself.
DerivedA.ClassProperty
DerivedB class_method called by instance.
DerivedB class_method called by instance.
DerivedB class_method called by Derived class itself.
Overrided static_method called by instance.
Overrided static_method called by Derived class itself.
0
False
True
{% endhighlight %}

### 参考资料：

- [static-and-class-methods][2]
- [What is the difference between @staticmethod and @classmethod in Python?][3]
- [Python @classmethod and @staticmethod for beginner?][4]

[1]: /2014/07/01/class-design-in-python.html
[2]: http://www.slideshare.net/papaJee/static-and-class-methods
[3]: http://stackoverflow.com/questions/136097/what-is-the-difference-between-staticmethod-and-classmethod-in-python
[4]: http://stackoverflow.com/questions/12179271/python-classmethod-and-staticmethod-for-beginner


