---
layout: post
title: "Qt中的信号与槽"
description: ""
category: [C++,Qt]
---

### Qt

#### [Qt][1]是一个开源的跨平台GUI框架，基于C++实现。Qt可以被用来开发Windows，Linux，MacOS，Android以及IOS等平台的应用开发。官网提供了一个独立的IDE：Qt Creator，针对Visual Studio也提供了Qt的插件以方便开发。我在[cnblogs的一篇文章中][2]介绍了如何在Windows平台编译Qt的静态库。

### Qt中的信号与槽

#### 信号与槽是Qt自己开发的通信机制，其也是Qt的核心机制，用于对象内部以及对象间的数据通信。在GUI开发中，我们需要对用户的操作事件进行相应的回应，并根据用户操作处理程序的数据，以及更新界面显示。通过信号与槽机制，我们可以非常方便高且高效地做到这些，而不是像使用MFC时定义很多回调函数。

#### 通过从QObject及其子类继承，新建的类型就可以拥有信号（signals）与槽（slots），我们需要做的，就是根据业务逻辑，将对象内部或者对象之间的信号与槽进行绑定（connect），然后在合适的时候发射（emit）相应的信号，这样与该信号绑定的槽就会被自动调用。声明信号的时候不用关心其将会和那些槽绑定，声明槽的时候也不用关心其将会和那些信号绑定。这样就可以做到解耦，提高设计的灵活性。一个信号可以和一个或者多个槽进行绑定，也可以和另一个信号绑定，对于前者，我们无法指定一个信号被发射后，和其绑定的槽执行的顺序；对于后者，一个信号被发射后，和其绑定的另一个信号也会马上被自动发射，在一些场合很有用。

### 信号（Signals）

#### 在继承自QObject或其子类的类型中定义signals的方式如下：

{% highlight c++%}
signals:
    void mySignal(void);
    void mySignal(const int &x);
{% endhighlight%}

* signals 是Qt的关键字，是Qt对标准C++的扩展，在signals块可以定义一组该类型的信号。
* 信号的返回值都为void。
* 信号的参数个数不限，可以有默认值。
* 信号函数可以重载。
* 信号没有函数体，在编译的时候，Qt会调用mete object compiler（moc）工具进行预编译，给QObject子类中的信号提供函数体。

### 槽（slots）

#### 槽函数是普通的成员函数，只是其参数不能有默认值。槽函数可以定义访问限制。
1. public slots - 该区块下面的 slots 函数作为普通成员函数是任何对象都可以访问的。
2. protected slots - 该区块下面的 slots 函数作为普通成员函数是只能被自身和其子类访问的。
3. private slots - 该区块下面的 slots 函数作为普通成员函数是类型自己的

#### 需要注意的是，这里的访问修饰符限定的是槽函数作为普通成员函数的访问级别，和标准C++定义的是相同的。而作为slot，无论是public，protected还是private修饰的槽函数都是public的，可以在外部和其他信号进行绑定。

#### 槽的声明方式如下：

{% highlight c++%}
protected slots:
    virtual void mySlot(const int &x);

private slots:
    void mySlot(const string &str);
{% endhighlight%}

* 槽可以是虚函数，可以实现多态。
* 槽一般情况下返回值为void。
* 槽函数的参数不能有默认值。

### 使用信号与槽

#### 下面，我们通过一个例子来说明如何在Qt项目中使用信号与槽。定义一个包含一个信号和一个纯虚函数槽的基类：

{% highlight c++%}
class Base : public QObject
{
    Q_OBJECT

signals:
    void mySignal(const string &str);

public slots:
    void mySlot(void);

protected slots:
    virtual void mySlot(const int &x) = 0;

};
{% endhighlight%}

#### 然后，继承上面的基类，实现基类中的纯虚函数槽，并定义一个私有的槽：

{% highlight c++%}
class Derived : public Base
{
    Q_OBJECT

public:
    Derived();
    ~Derived(){}

// override the slot method to bring us polymorphysm.
protected slots:
    virtual void mySlot(const int &x);

private slots:
    void mySlot(const string &str);
};
{% endhighlight%}

#### 定义另外一个QObject的子类，只包含信号和一个发射所有信号的公共成员方法：

{% highlight c++%}
class Another : public QObject
{
    Q_OBJECT

public:
    Another(){}
    ~Another(){}
    void EmitAllSignals(void);

signals:
    void mySignal(void);
    void mySignal(const int &x);
};
{% endhighlight%}

#### 实现上面类型中的成员方法：

{% highlight c++%}
void Base::mySlot(void)
{
    cout << "Base::mySlot(void)" << endl;
}

void Derived::mySlot(const string &str)
{
    cout << "Derived::mySlot(const std::string & str), \nstr = " << str << endl;
}

void Derived::mySlot(const int &x)
{
    cout << "Derived::mySlot(const int &x), x = " << x << endl;
}

Derived::Derived()
{
    connect(this, SIGNAL(mySignal(const string &)),
            SLOT(mySlot(const string &)));
    emit mySignal("Derived signal mySignal(const std::string & str) emmited\n");
}

void Another::EmitAllSignals(void)
{
    emit mySignal();
    emit mySignal(100);
}
{% endhighlight%}

#### 最后，测试一下：

{% highlight c++%}
#include <QCoreApplication>
#include "sample.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    Base *base = new Derived();
    Another *another = new Another();

    QObject::connect(another, SIGNAL(mySignal(void)),
                     base, SLOT(mySlot(void)));
    QObject::connect(another, SIGNAL(mySignal(const int &)),
                     base, SLOT(mySlot(const int &)));

    another->EmitAllSignals();

    // disconnect with a certain signal
    another->disconnect(SIGNAL(mySignal(void)));
    another->EmitAllSignals();

    // disconnect with all signals of a certain object
    another->disconnect(base);
    another->EmitAllSignals();

    delete base;
    delete another;

    return a.exec();
}
{% endhighlight%}

* 绑定信号与槽的方式为：
{% highlight c++%}
bool QObject::connect ( const QObject * sender, const char * signal, 
         const QObject * receiver, const char * member ) [static]
{% endhighlight%}

* 取消绑定的方式为：

{% highlight c++%}
bool QObject::disconnect ( const QObject * sender, const char * signal, 
         const Object * receiver, const char * member ) [static]
{% endhighlight%}

* 绑定和取消绑定的方法有很多重载实现，具体可以参考[这篇文章][3]

#### 本文介绍了Qt中的信号与槽已经它们的使用方法，它们的实现机制依赖与Qt对C++的扩展和moc工具的预编译。

[1]: http://qt-project.org/
[2]: http://www.cnblogs.com/rwang/p/3399996.html
[3]: http://www.ibm.com/developerworks/cn/linux/guitoolkit/qt/signal-slot/


