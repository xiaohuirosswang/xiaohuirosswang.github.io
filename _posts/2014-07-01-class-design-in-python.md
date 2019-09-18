---
layout: post
title: "Python中的类型设计"
description: ""
category: [Python,OOD]
---

### Python中的类

#### 和C++等静态强类型语言不通，Python是动态类型语言，天然支持动态绑定，在Python中一切皆对象，类也一样。

#### 在Python中定义一个类，需要明确下面几个概念

* 类属性 - 需要在类中显式定义，可以通过类方法，类实例方法读取，只能通过类方法以及类本身调用来修改，当然也可以在类静态方法中通过类本身调用修改。
* 类实例属性 - 可以在类外动态定义，也可以在类实例方法中定义。
* 类方法 - 通过类名访问的方法。（非私有方法）
* 类实例方法 - 通过类实例访问的方法 （非私有方法）
* 类静态方法 - 不需要接受self或者cls参数，通过类名访问

#### 另外需要提到的是，类属性，类实例属性以及类方法，类实例方法，类静态方法都可以定义为私有方法，通过在名称前加两个下划线__实现。

#### 下面，我们通过一个例子来看看这些概念的使用方法

1. 定义Car类

{% highlight python %}
class Car:
    """A base class for all cars"""

    # A class attribute
    engine = "NO2"

    def __init__(self, name="Unknown", price=0):
        """Initialize the private class attributes"""

        # Here are 2 class instance attributes
        # name is a public attribute, while __price is a private attribute
        self.name = name
        self.__price = price

    # A class method
    @classmethod
    def explain_self(cls):
        """Print the essential information about the class"""

        print "The engine of car is", cls.engine

    # A static method
    @staticmethod
    def change_engine(new_engine):
        """Change the engine type"""

        Car.engine = new_engine  # Need to use "class.attribute"

    # A class instance method
    def about_me(self):
        """Print the essential information about the class"""

        print "This car is", self.name, "whose price is", self.__price


{% endhighlight %}

2. 通过下面的方法调用：

{% highlight python %}
if __name__ == "__main__":

    c1 = Car("AUDI", 200000)
    c1.about_me()
    print "Before change, the engine of", c1.name, "is", c1.engine
    c1.engine = "Gas"
    print "After change, the engine of", c1.name, "is", c1.engine

    Car.explain_self()
    print "Before change, the engine of Car is", Car.engine
    Car.change_engine("Gas and NO2")
    print "After change, the engine of Car is", Car.engine

    c2 = Car("BMW", 300000)
    c2.about_me()
    print "The engine of BMW is", Car.engine

    print Car.__dict__     #  对象的说明文档信息
    print Car.__doc__      #  对象所包含的元素信息
    print Car.__module__   #  对象的模块信息

    print c1.__dict__
{% endhighlight %}

输出为：

{% highlight python %}
This car is AUDI whose price is 200000
Before change, the engine of AUDI is NO2
After change, the engine of AUDI is Gas
The engine of car is NO2
Before change, the engine of Car is NO2
After change, the engine of Car is Gas and NO2
This car is BMW whose price is 300000
The engine of BMW is Gas and NO2
{'engine': 'Gas and NO2', '__module__': '__main__', 'change_engine': <staticmethod object at 0x00000000026C4A98>, 'about_me': <function about_me at 0x00000000026C8B38>, 'explain_self': <classmethod object at 0x00000000026C4A68>, '__doc__': 'A base class for all cars', '__init__': <function __init__ at 0x00000000026C89E8>}
A base class for all cars
__main__
{'engine': 'Gas', '_Car__price': 200000, 'name': 'AUDI'}
{% endhighlight %}


#### 完整代码见[gist][1]

[1]: https://gist.github.com/xhrwang/93d6a10baaf7012271bc


