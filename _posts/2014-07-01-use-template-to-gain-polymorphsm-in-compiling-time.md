---
layout: post
title: "模板基类与编译期多态"
description: ""
category: [C++]
---

### 引子

#### 知乎上有个[关于模板基类的问题][1]，很有意思，正如答题的知友们提到的，这个问题可以引申到[curiously recurring template pattern][2]。

#### 模板在C++标准库和其他一些被广泛应用的第三方库（比如Boost）中应用的非常广泛，虽然影响到了代码的可读性，但是极大减少了代码量，提供了灵活设计的可能。

#### 关于模板元编程，[vczh][3]牛牛在[这篇文章][4]中提到学完数学家们发明的Haskell之后：
> ##### 于是回过头来，模板元编程也就变成一个很自然的东西了。你把模板元编程看成是一门语言，把“类型”本身看成是一个巨大的带参数enum的一部分（scala叫case type），于是类型的名字就变成了值，那么模板元编程的技巧，其实就是对类型进行变换、操作和计算的过程。

#### 本文不讨论Haskell，笔者看完上面的文章后，配置好Haskell的开发环境，了解过一些Haskell的语法，但远未入门。这里只是想在知乎上那个问题的基础上，讨论一下模板基类在实现编译器多态时的用法。

### 编译期多态

#### 在C++项目的设计开发中，我们用到的比较多的，是在含有virtual方法的基类基础上，通过继承和override而获得的运行时多态。也就是说，在运行时，基类的引用或者指针可以根据传递的派生类类型决定调用那个方法，从而达到动态绑定的目的。但是这种用法，程序需要维护一个指向虚函数表的虚函数指针，一定程度上会影响到性能。

#### 利用模板技术，我们可以把运行时进行的绑定前移到编译时，这样编译后的程序体积会相对变大，但是性能上没有损失，在性能敏感的场合比如嵌入式开发中有一定的意义。接下来，我们通过代码来看看如何通过设计模板基类来做到这一点。

1. 定义模板基类，“(static_cast<T*>(this)” 会将基类转化为派生类，从而调用派生类的方法。

{% highlight C++ %}
template<typename T>
class Base
{
public:
	string GetDisplayStr(){return "I am in Base.";}
	void SaySomething(void){cout << (static_cast<T*>(this))->GetDisplayStr() << endl;}

private:

};
{% endhighlight %}


2. 定义两个派生类，从上面的模板基类继承。

{% highlight C++ %}
class DerivedA : public Base<DerivedA>
{
    //no additional code here
};

class DerivedB : public Base<DerivedB>
{
public:
	static string GetDisplayStr(){return "I am in DerivedB.";} //re-define the method
};
{% endhighlight %}

3. 定义一个模板方法

{% highlight C++ %}
template<typename T>
void CallSaySomething(Base<T> *p)
{
	p->SaySomething();
}
{% endhighlight %}

4. 通过下面的代码进行验证

{% highlight C++ %}
int main()
{
	Base<DerivedA> *ba = new DerivedA();
	Base<DerivedB> *bb = new DerivedB();
	CallSaySomething(ba);
	CallSaySomething(bb);
	delete ba;
	delete bb;

	return 0;
}
{% endhighlight %}

输出为：

{% highlight C++ %}
I am in Base.
I am in DerivedB.
{% endhighlight %}

#### 可以看到我们没有用虚方法来实现多态，而且绑定在编译器就已经完成了。完整代码见[gist][5]

[1]: http://www.zhihu.com/question/24287316
[2]: http://en.wikipedia.org/wiki/Curiously_recurring_template_pattern
[3]: http://www.zhihu.com/people/geniusvczh
[4]: http://www.cppblog.com/vczh/archive/2013/03/24/198769.html
[5]: https://gist.github.com/xhrwang/b3cd8ddc3dfb90ebcba6


