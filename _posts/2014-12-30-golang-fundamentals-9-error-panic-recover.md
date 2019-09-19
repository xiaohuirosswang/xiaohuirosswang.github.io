---
layout: post
title: "Golang 语言基础之九： error, panic, recover"
description: ""
category: [Golang]
---

Golang 语言基础系列：

- [Golang 语言基础之一： type, variable, constant](/golang-fundamentals-1-types-variables-constants.html)
- [Golang 语言基础之二： for, ifelse, switch](/golang-fundamentals-2-for-ifelse-switch.html)
- [Golang 语言基础之三： array, slice](/golang-fundamentals-3-array-slice.html)
- [Golang 语言基础之四： map, range](/golang-fundamentals-4-map-range.html)
- [Golang 语言基础之五： function](/golang-fundamentals-5-function.html)
- [Golang 语言基础之六： string, pointer](/golang-fundamentals-6-string-pointer.html)
- [Golang 语言基础之七： struct, method](/2014/12/28/golang-fundamentals-7-struct-method.html)
- [Golang 语言基础之八： interface](/golang-fundamentals-8-interface.html)
- [Golang 语言基础之九： error, panic, recover](/golang-fundamentals-9-error-panic-recover.html)
- [Golang 语言基础之十： goroutine, channel](/golang-fundamentals-10-goroutine-channel.html)

很多编程语言都有异常处理机制，C++、C# 和 JAVA 中有 `try{...}catch(...){...}` 和 `throw new SomeException()` 这样的语法来捕获、处理以及抛出异常，C# 和 JAVA 还支持 `try{...}catch(...){...}finally{...}` 这样的语法来保证异常发生的时候执行 `finally` 代码块中的逻辑来做些运行时清理工作。Python 里面也有类似下面的异常机制：

{% highlight python %}
try:
    doSomthing()
except someError:
    handleException()
    raise
{% endhighlight %}

在实际的项目中，对于异常的 Best Practice 很多，在使用不同的语言开发不同类型的程序时，有不同的建议。[Google C++ Style][1] 中提到 Google 内部的 C++ 代码中不使用异常，社区也有很多关于异常的讨论：

- [Exception-Safe Coding in C++][2]
- [Best Practices for Exceptions][3]
- [Google C++ style guide's No-exceptions rule; STL?][4]
- [Google C++ coding style, no exceptions rule. What about multithreading?][5]

作为一门相对来说很新的语言，Golang 中没有使用传统的 `try...catch` 类似的异常处理机制，而是提供了 `panic` 和 `recover` 函数来处理所谓的 `运行时异常`，也就是 Google 所称的 __错误处理机制__。配合 `defer` 语句和 `error` 接口可以让开发者非常灵活地处理运行时的错误和异常。[官方博客的这篇文章][6] 对此做了详细解释。 

从命名可以看出，Google 肯定不希望开发者在代码中随便使用 `panic`，我们知道在风险控制中，有所谓 __已知的未知__ 和 __未知的未知__，在 Golang 程序设计中，对于前者，我们可以通过预先开发的代码分支来处理；对于后者就比较 tricky：

- 如果项目中的代码、使用的标准库以及第三方库在运行时内部捕获了异常并通过合适的 `error` 对象返回给调用者，那我们可以尽量少甚至可以不用 `panic` 函数。
- 如果无法保证上面的情况，那为了确保程序在运行时不会因为 __未知的未知__ 导致崩溃，那 `panic` 函数的使用可能不得不加在任何需要的地方。

所以，当我们开发 Library 时，`panic` 机制的使用或许不可避免，而在开发应用程序的时候，对于比如像 IO 异常、数据库连接异常等运行时可能会出现的异常情况，为了保证应用程序的健壮性，我们无法完全避免使用 `panic`。但是我们应该认识到，`恐慌` 是我们和计算机都不希望看到的，应该在设计开发的时候充分考虑使用场景可能出现的情况，处理好 __已知的未知__，在确定需要的地方使用 `panic` 机制。

## Error

Golang 通过支持 `多返回值` 让在运行时返回详细的错误信息给调用者变得非常方便。我们可以在编码中通过实现 `error` 接口类型来生成错误信息，[error 接口的定义][7] 如下：

{% highlight go %}
type error interface {
    Error() string
}
{% endhighlight %}

还是通过下面的例子来看看：

{% highlight go %}
package main

import (
	"fmt"
)

// 定义一个 DivideError 结构
type DivideError struct {
	dividee int
	divider int
}

// 实现 	`error` 接口
func (de *DivideError) Error() string {
	strFormat := `
	Cannot proceed, the divider is zero.
	dividee: %d
	divider: 0
`
	return fmt.Sprintf(strFormat, de.dividee)
}

// 定义 `int` 类型除法运算的函数
func Divide(varDividee int, varDivider int) (result int, errorMsg string) {
	if varDivider == 0 {
		dData := DivideError{
			dividee: varDividee,
			divider: varDivider,
		}
		errorMsg = dData.Error()
		return
	} else {
		return varDividee / varDivider, ""
	}

}

func main() {

	// 正常情况
	if result, errorMsg := Divide(100, 10); errorMsg == "" {
		fmt.Println("100/10 = ", result)
	}
	// 当被除数为零的时候会返回错误信息
	if _, errorMsg := Divide(100, 0); errorMsg != "" {
		fmt.Println("errorMsg is: ", errorMsg)
	}

}
{% endhighlight %}

将上面的代码存入源文件 error.go 并使用 `go run error.go` 可以看到下面的输入：

{% highlight text %}
100/10 =  10
errorMsg is:  
	Cannot proceed, the divider is zero.
	dividee: 100
	divider: 0
{% endhighlight %}

注释很清楚了，就不多说了。

## Panic 和 recover

[官方文档关于 panic 和 recover 的定义][8] 如下：

{% highlight go %}
func panic(interface{})
func recover() interface{}
{% endhighlight %}

`panic` 和 `recover` 是两个内置函数，用于处理 [run-time panics][9] 以及程序中自定义的错误。

当执行一个函数 `F` 的时候，如果显式地调用 `panic` 函数或者一个 [run-time panics][9] 发生时，`F` 会结束运行，所有 `F` 中 `defer` 的函数会按照 FILO 的规则被执行。之后，`F` 函数的调用者中 `defer` 的函数再被执行，如此一直到最外层代码。这时，程序已经被中断了而且错误也被一层层抛出来了，其中包括 `panic` 函数的参数。当前被中断的 `goroutine` 被称为处于 `panicking` 状态。由于 `panic` 函数的参数是空接口类型，所以可以接受任何类型的对象：

{% highlight go %}panic(42)
panic(42)
panic("unreachable")
panic(Error("cannot parse"))
{% endhighlight %}

`recover` 函数用来获取 `panic` 函数的参数信息，只能在延时调用 `defer` 语句调用的函数中直接调用才能生效，如果在 `defer` 语句中也调用 `panic` 函数，则只有最后一个被调用的 `panic` 函数的参数会被 `recover` 函数获取到。如果 `goroutine` 没有 panic，那调用 `recover` 函数会返回 `nil`。

关于错误处理，[Effective Go][10] 中做了一些很好的解释。

看一个例子：

{% highlight go %}
package main

import (
	"fmt"
)

// 最简单的例子
func SimplePanicRecover() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("Panic info is: ", err)
		}
	}()
	panic("SimplePanicRecover function panic-ed!")
}

// 当 defer 中也调用了 panic 函数时，最后被调用的 panic 函数的参数会被后面的 recover 函数获取到
// 一个函数中可以定义多个 defer 函数，按照 FILO 的规则执行
func MultiPanicRecover() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("Panic info is: ", err)
		}
	}()
	defer func() {
		panic("MultiPanicRecover defer inner panic")
	}()
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("Panic info is: ", err)
		}
	}()
	panic("MultiPanicRecover function panic-ed!")
}

// recover 函数只有在 defer 函数中被直接调用的时候才可以获取 panic 的参数
func RecoverPlaceTest() {
	// 下面一行代码中 recover 函数会返回 nil，但也不影响程序运行
	defer recover()
	// recover 函数返回 nil
	defer fmt.Println("recover() is: ", recover())
	defer func() {
		func() {
			// 由于不是在 defer 调用函数中直接调用 recover 函数，recover 函数会返回 nil
			if err := recover(); err != nil {
				fmt.Println("Panic info is: ", err)
			}
		}()

	}()
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("Panic info is: ", err)
		}
	}()
	panic("RecoverPlaceTest function panic-ed!")
}

// 如果函数没有 panic，调用 recover 函数不会获取到任何信息，也不会影响当前进程。
func NoPanicButHasRecover() {
	if err := recover(); err != nil {
		fmt.Println("NoPanicButHasRecover Panic info is: ", err)
	} else {
		fmt.Println("NoPanicButHasRecover Panic info is: ", err)
	}
}

// 定义一个调用 recover 函数的函数
func CallRecover() {
	if err := recover(); err != nil {
		fmt.Println("Panic info is: ", err)
	}
}

// 定义个函数，在其中 defer 另一个调用了 recover 函数的函数
func RecoverInOutterFunc() {
	defer CallRecover()
	panic("RecoverInOutterFunc function panic-ed!")
}

func main() {
	SimplePanicRecover()
	MultiPanicRecover()
	RecoverPlaceTest()
	NoPanicButHasRecover()
	RecoverInOutterFunc()
}
{% endhighlight %}

将上面的代码存入源文件 panic.go 并使用 `go run panic.go` 可以看到下面的输入：

{% highlight text %}
Panic info is:  SimplePanicRecover function panic-ed!
Panic info is:  MultiPanicRecover function panic-ed!
Panic info is:  MultiPanicRecover defer inner panic
Panic info is:  RecoverPlaceTest function panic-ed!
recover() is:  <nil>
NoPanicButHasRecover Panic info is:  <nil>
Panic info is:  RecoverInOutterFunc function panic-ed!
{% endhighlight %}

需要注意的地方都在上面的例子中有体现，注意注释和函数名字。

## 参考资料

- [The Go Programming Language](http://golang.org/cmd/go/)
- [学习 Go 语言  中文版](http://mikespook.com/learning-go/)
- [Go in Action  中文版](https://github.com/astaxie/Go-in-Action)
- [The way to Go 中文版](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/02.2.md)
- [Go by Example](https://gobyexample.com/hello-world)
- [Organizing Go code](https://talks.golang.org/2014/organizeio.slide#1)
- [Testing Techniques](https://talks.golang.org/2014/testing.slide#1)
- [Go 语言分享](http://www.jiagoushi.me/index.php/archives/43/)
- [Go 学习笔记](https://github.com/qyuhen/book)
- [Go 语言简介](http://coolshell.cn/articles/8460.html)
- [Tony Bai 的博客](http://tonybai.com/)

[1]: http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Exceptions
[2]: http://www.exceptionsafecode.com/
[3]: http://msdn.microsoft.com/en-us/library/seyhszts(v=vs.110).aspx
[4]: http://stackoverflow.com/questions/5184115/google-c-style-guides-no-exceptions-rule-stl
[5]: http://stackoverflow.com/questions/19073441/google-c-coding-style-no-exceptions-rule-what-about-multithreading
[6]: http://blog.golang.org/error-handling-and-go
[7]: https://golang.org/doc/effective_go.html#errors
[8]: http://golang.org/ref/spec#Handling_panics
[9]: http://golang.org/ref/spec#Run_time_panics
[10]: https://golang.org/doc/effective_go.html#errors


