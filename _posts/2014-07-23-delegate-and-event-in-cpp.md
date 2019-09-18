---
layout: post
title: "委托和事件的C++实现"
description: ""
category: [C++]
---

### 委托和事件

#### C#语言中的委托和事件使用非常方便，随着语言的进化，后来加入了匿名方法，lambda表达式以及泛型委托，给程序设计提供了很大的灵活性和可读性。

#### C++中提供了函数指针和成员方法指针，没有委托和事件。在程序设计中，特别是界面应用程序开发中，我们通常会用到MVC模式，将界面部分独立出来成为View模块，在Controller模块中通过回调函数的方式对View中的控件状态进行更新。但是由于缺少对委托和事件的直接支持，设计的时候不容易很清晰得进行模块间解耦。有些界面框架比如Qt，引入了信号与槽机制，其实是以拓展语言的方式加入了委托和事件。而正是由于拓展了语言（signals和slots关键字，还有connect的时候用到的对应的宏），Qt实现了一个预编译器meta object compiler，在使用C++编译器编译之前，对代码进行预编译，自动生成标准C++支持的代码。

#### Qt，FastDelegate等开源框架给C++带来了对委托和事件的支持，对于我们开发者来说，理解它们的机制并能正确运用是一方面，理解如何自己使用C++实现委托和事件是另一方面，对我们熟悉C++本身，使用模板编程都有意义。

#### 在C++中实现委托，需要用到函数指针和模板类以及模板方法，我们先从函数指针开始，并在后面的代码示例中解释模板类和模板函数的使用方式和必要性。

### 函数指针

{% highlight c++ %}
typedef int (*pFunction)(int,char)=NULL;
typedef int (SampleClass::*pMemberFunction)(int,char)=NULL;
typedef int (SampleClass::*pConstMemberFunction)(int,char) const=NULL;
{% endhighlight %}

1. 上面定义了三个函数指针，返回值都为int类型，都有两个参数，依次为int和char类型。为了可读性，我们嫁了typedef关键字。
2. 第一行定义了一个一般的函数指针。
3. 第二行定义了一个非const的成员方法指针。
4. 第三行定义了一个const的成员方法指针。

#### 关于函数指针的赋值和调用，在下面的例子中会演示和解释。

### C++实现委托和事件

#### 还是通过一个demo来练习一下，这里，我们实现一个类似于C#中的泛型委托Action<T1, T2>的委托。

* 定义基类

{% highlight c++ %}
//Define a template class, the 2 typenames are the types of the parameters for the delegate function
template<typename T1, typename T2>
class ActionBase
{
public:
    virtual ~ActionBase(){}
    virtual void operator()(T1,T2) = 0;
    virtual bool eq( unsigned long ) = 0;
    virtual bool eq( void*, unsigned long){ return false;}
};
{% endhighlight %}

1. 重载操作符()，在其实现中调用回调函数，并传入T1，T2类型的参数。
2. eq方法用来对函数指针进行相等比较，在后面的事件类型中，我们会维护一个包含ActionBase类型指针的vector，保存对该事件绑定的委托对象。

* 定义一个ActionBase的派生类，该类型使用成员方法指针。

{% highlight c++ %}
//Action4MemberMethod is using a member method pointer
template<typename TMemberMethodProvider,typename T1,typename T2>
class  Action4MemberMethod : public ActionBase<T1,T2>
{
private:
    //A member method pointer using the generic parameters.
    typedef void (TMemberMethodProvider::*pFun)(T1,T2);
    TMemberMethodProvider*    pCLS;
    pFun    pf;

public:
    Action4MemberMethod(TMemberMethodProvider* p, pFun pf)
    {
        pCLS = p;
        this->pf = pf;
    }

    virtual void operator()(T1 t1, T2 t2)
    {
        // Note how to call a function via a function pointer
        if(pCLS)
            (pCLS->*pf)(t1, t2);
    }

    //Determine whether current member method pointer is the same as the given one or not
    virtual bool eq ( TMemberMethodProvider* ps, pFun pf )
    {
        return eq ( (void*)ps, (unsigned long)&pf);
    }

    virtual bool eq (void* p,unsigned long ul)
    {
        return *(unsigned long*)&this->pf == ul && p == pCLS;
    }

    virtual bool eq (unsigned long)
    {
        return false;
    }
};
{% endhighlight %}

* 定义另一个ActionBase的派生类，使用一般函数指针。

{% highlight c++ %}
//Action4RegularFunction is using a regular function pointer
template<typename T1,typename T2>
class  Action4RegularFunction : public ActionBase<T1,T2>
{
private:
    typedef void (*pFun)(T1,T2);
    pFun pf;

public:
    Action4RegularFunction(  pFun p)
    {
        this->pf = p;
    }

    virtual void operator()(T1 t1, T2 t2)
    {
        // Note how to call a function via a function pointer
        this->pf(t1, t2);
    }

    //Determine whether the current function pointer is the same as the given one or not
    virtual bool eq (unsigned long ul)
    {
        return *(unsigned long*)&(this->pf) == ul;
    }

    virtual bool eq (void* p,unsigned long ul)
    {
        return false;
    }
};
{% endhighlight %}

* 定义一个事件类

{% highlight c++ %}
template<typename T1, typename T2>
class EventHandler
{
private:
    typedef vector<ActionBase<T1,T2>*> vFBase;
    typedef typename vFBase::iterator   vFBaseI;
    vFBase  vF;

public:
    EventHandler()
    {
    }

    template<typename CLS>
    void bind( CLS* p, void (CLS::*pFun)(T1,T2))
    {
        ActionBase<T1,T2>* pf = new Action4MemberMethod<CLS,T1,T2>(p,pFun);
        vF.push_back(pf);
        printf("Bind function %d to this event\n", *(unsigned long*)&pFun);
        printf("Now there are %d functions binded to this event\n", vF.size());
    }

    void bind( void (*pFun)(T1,T2) )
    {
        ActionBase<T1,T2>* pf = new Action4RegularFunction<T1,T2>(pFun);
        vF.push_back(pf);
        printf("Bind function %d to this event\n", *(unsigned long*)&pFun);
        printf("Now there are %d functions binded to this event\n", vF.size());

    }

    template<typename CLS>
    void unbind( CLS* p, void (CLS::*pFun)(T1,T2))
    {
        vFBaseI i = vF.begin();
        while ( i != vF.end() )
        {
            if ( (*i)->eq(p, *(unsigned long*)&pFun))
            {
                printf("Unbind function %d to this event\n", *(unsigned long*)&pFun);
                delete *i;
                vF.erase(i);
                printf("Now there are %d functions binded to this event\n", vF.size());
                break;
            }
            i++;
        }
    }

    void unbind( void (*pFun)(T1,T2))
    {
        vFBaseI i = vF.begin();
        while ( i != vF.end() )
        {
            if ( (*i)->eq(*(unsigned long*)&pFun))
            {
                printf("Unbind function %d to this event\n", *(unsigned long*)&pFun);
                delete *i;
                vF.erase(i);
                printf("Now there are %d functions binded to this event\n", vF.size());
                break;
            }
            i++;
        }
    }

    //Trigger the event, and execute the binded delegated functions one by one.
    void doEvent(T1 t1, T2 t2)
    {
        if(vF.size() == 0)
        {
            printf("No function binded.\n");
            return;
        }
        vFBaseI i = vF.begin();
        while( i != vF.end() )
        {
            printf("Executing binded function.\n");
            (*(*i))(t1, t2);
            i++;
        }
        printf("All binded functions executed!\n");
    }
};
{% endhighlight %}

1. bind方法用于将委托对象绑定到该事件，通过将指向ActionBase的派生类对象的指针保存如ActionBase*类型的vector实现。
2. unbind方法用于将已经绑定的委托对象移除，这里用到了委托对象的eq方法，通过比较函数指针的内存地址确定移除哪一个委托对象。
3. doEvent会触发事件，并依次执行绑定到该事件的委托对象中函数指针指向的方法。

* 好了，测试一下：

{% highlight c++ %}
class Test
{
public:
    void print(  int n, char* p)
    {
        printf("A member function in Test:\n========Paras: %d, %s\n", n,p);
    }
};

void print( int n, char* p)
{
    printf("A regular global function:\n========Paras: %d, %s\n", n,p );
}

int main()
{
    EventHandler<int,char*> event;
    Test a;
    // Bind 2 functions to the event
    event.bind(&a, &Test::print);
    event.bind(&print);
    // Trigger the event
    event.doEvent(10, "Fire an event with 10");

    // Unbind all functions binded to the event
    event.unbind(&a, &Test::print);
    event.unbind(&print );
    // Trigger the event again to see what will happen
    event.doEvent(11, "Fire an event with 11");
    return 0;
}
{% endhighlight %}

#### 运行后的输出为：

	Bind function 16258109 to this event
	Now there are 1 functions binded to this event
	Bind function 16258144 to this event
	Now there are 2 functions binded to this event
	Executing binded function.
	A member function in Test:
	========Paras: 10, Fire an event with 10
	Executing binded function.
	A regular global function:
	========Paras: 10, Fire an event with 10
	All binded functions executed!
	Unbind function 16258109 to this event
	Now there are 1 functions binded to this event
	Unbind function 16258144 to this event
	Now there are 0 functions binded to this event
	No function binded.

### 后记

#### 这里我们只是实现了类似于C#中Action<T1,T2>的泛型委托，对于想Func，Predicate等，没有涉及到，而且只能接收两个参数，从中可以看到，语言特性的Flavor是不容易加入的，而且我们的代码也没有做线程安全的设计，对于事件类型中的vector的访问没有做线程同步。所以，这个例子只能帮助我们理解委托和事件的工作机制，如果需要完整的工程级别可用的C++实现，可以参考Boost和FastDelegate，Qt的信号与槽也可以，只是Qt涉及到moc预编译，不是纯粹C++语言开发层面的了。


