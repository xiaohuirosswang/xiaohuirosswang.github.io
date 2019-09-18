---
layout: post
title: "Golang 语言基础之五： function"
description: ""
category: [Golang]
---

Golang 语言基础系列：

- [Golang 语言基础之一： type, variable, constant](/2014/12/22/golang-fundamentals-1-types-variables-constants.html)
- [Golang 语言基础之二： for, ifelse, switch](/2014/12/23/golang-fundamentals-2-for-ifelse-switch.html)
- [Golang 语言基础之三： array, slice](/2014/12/23/golang-fundamentals-3-array-slice.html)
- [Golang 语言基础之四： map, range](/2014/12/25/golang-fundamentals-4-map-range.html)
- [Golang 语言基础之五： function](/2014/12/26/golang-fundamentals-5-function.html)
- [Golang 语言基础之六： string, pointer](/2014/12/27/golang-fundamentals-6-string-pointer.html)
- [Golang 语言基础之七： struct, method](/2014/12/28/golang-fundamentals-7-struct-method.html)
- [Golang 语言基础之八： interface](/2014/12/29/golang-fundamentals-8-interface.html)
- [Golang 语言基础之九： error, panic, recover](/2014/12/30/golang-fundamentals-9-error-panic-recover.html)
- [Golang 语言基础之十： goroutine, channel](/2014/12/31/golang-fundamentals-10-goroutine-channel.html)

## 函数

`function` 是 Golang 的核心，它的定义方式为：

	func (variable Type) funcName(var1 Type1, var2 Type2, variabicPara ...TypeX) (ret1 ReturnType1, ret2 ReturnType2) {   }
	 ^        ^      ^      ^       ^     ^     ^     ^                 ^          ^        ^        ^        ^         ^
	 1        2      3      4       5     6     7     8                 9          10       11       12       13        14

1. 使用 `func` 关键字来定义函数。
2. 如果定义时有 `(variable Type)` 的内容，`variable` 表示函数所属类型的对象，如果函数体中不需要用到则可以省略。
3. 如果定义时有 `(variable Type)` 的内容，`Type` 表示函数所属的类型，`Type` 本身也可以是一个类型的指针，比如 `struct` User 的指针 `*User`。
4. 函数的名称。
5. 函数可以有 `0` 个或者多个形参，函数第一个形参的名字。
6. 函数第一个形参的类型。
7. 函数第二个形参的名字。
8. 函数第二个形参的类型。
9. `variabicPara ... TypeX` 表示可以有可变参数，也就是是零个或者多个同一类型 `TypeX` 的形参，实参传递的时候放入一个名为 `variabicPara` 的 `slice` 对象中。
10. 函数可以有多个返回值，第一个返回值的名称。（函数可以指定 `命名返回值`，也就是说返回值名称在定义函数时确定，并初始化为各自类型的零值。它们在函数内部的使用方式和函数传递的实参是一样的。）
11. 第一个返回值的类型。
12. 第二个返回值的名称。
13. 第二个返回值的类型。
14. 函数体，在里面可以定义函数的行为。

__注意__，根据下面 [官方文档关于分号的说明][3]，Golang 编译器会在源代码中自动添加分号 `;`，如果 `{` 单独放在一行的开头，其前面可能会被加入 `;`，造成编译错误。

{% highlight text %}
The formal grammar uses semicolons ";" as terminators in a number of productions. Go programs may omit most of these semicolons using the following two rules:
1. When the input is broken into tokens, a semicolon is automatically inserted into the token stream at the end of a non-blank line if the line's final token is
	an identifier
	an integer, floating-point, imaginary, rune, or string literal
	one of the keywords break, continue, fallthrough, or return
	one of the operators and delimiters ++, --, ), ], or }
2. To allow complex statements to occupy a single line, a semicolon may be omitted before a closing ")" or "}".
{% endhighlight %}

[这个规则适用于其他流程控制语句（for, ifelse, switch, select）][4]:

> 注意：无论任何时候，你都不应该将一个控制结构（(if、for、switch或select）的左大括号放在下一行。如果这样做，将会在大括号的前方插入一个分号，这可能导致出现不想要的结果。

而对于 `struct`，`interface` 等类型，`gofmt` 会自动将左大括号 `{` 调整到原型声明那一行，所以无需操心。

按照本系列的惯例，还是从实际使用的例子开始：

{% highlight go %}
package main

import "fmt"

// 定义无参数也无返回值的函数
func NoParaNoReturn() {
	fmt.Println("I am a function with no parameter and return nothing.")
}

// 定义有参数但是无返回值的函数
func WithParasButNoReturn(intPara1, intPara2 int, strPara string) {
	fmt.Println("My parameters are: ", intPara1, intPara2, strPara)
}

// 定义有参数也有返回值的函数
func WithpParasAndReturn(intPara int) (int, int) {
	return intPara, intPara * 2
}

//定义有参数也有含命名返回值的函数，return 的时候会默认返回命名返回值 doubledInput
func WithpParasAndNamedReturn(intPara int) (doubledInput int) {
	fmt.Println("Initial value of doubledInput of int type is: ", doubledInput)
	doubledInput = intPara * 2
	return
}

// 定义有参数也有含命名返回值的函数，如果内部声明了和命名返回值同名的局部变量，需要显式返回命名返回值
func WithpNamedReturnAndInnerVar(intPara int) (output int) {
	{
		var output = 100
		output = intPara * 2
		return output
	}
}

// 定义有 可变参数 的函数
func WithVariabicPara(prefix string, paras ...int) {
	tempInt := 0
	for _, para := range paras {
		tempInt += para
	}
	fmt.Printf("%s, The sum of paras is: %d\n", prefix, tempInt)
}

// 定义含有闭包或者说匿名函数的函数
func WithClosure(intPara int) func() {
	returnFunc := func() {
		fmt.Println("Input is: ", intPara)
	}
	return returnFunc
}

// 定义含有 延时代码 的函数
func DelayCode() {
	defer fmt.Println("Defer: I will be executed at last")
	defer fmt.Println("Defer: I will be executed at first")

	fmt.Println("I will be executed before defer")
}

// 定义含有回调函数的函数
func UseCallback(myFunc func()) {
	myFunc()
}

// 定义递归函数
func UseRecursion(num int) int {
	if num <= 0 {
		return 0
	} else if num == 1 || num == 2 {
		return 1
	} else {
		return UseRecursion(num-1) + UseRecursion(num-2)
	}
}

// 定义结构体
type User struct {
	name string
}

// 定义属于结构体 User 的方法
func (user *User) NameChangedTo(new string) {
	user.name = new
}

// 定义属于结构体 User 的另一个方法
func (user *User) GetName() string {
	return user.name
}

func main() {
	// 调用无参数也无返回值的函数
	NoParaNoReturn()

	// 调用有参数但是无返回值的函数
	WithParasButNoReturn(1, 2, "OK")

	// 调用有参数也有返回值的函数
	input, output := WithpParasAndReturn(10)
	fmt.Printf("The double of %d is: %d\n", input, output)

	// 调用有参数也有含命名返回值的函数
	fmt.Println("The double of 10 is: ", WithpParasAndNamedReturn(10))

	// 调用有参数也有含命名返回值以及同名局部变量的函数
	fmt.Println("The double of 10 is: ", WithpNamedReturnAndInnerVar(10))

	// 调用 可变参数 的函数
	WithVariabicPara("Test", 1, 2, 3)

	// 调用递归函数
	fmt.Printf("The %d num of Fabinaci is: %d\n", 5, UseRecursion(5))

	// 调用含有闭包的函数
	WithClosure(100)()

	// 调用含有 DelayCode 的函数
	DelayCode()

	// 调用含有回调函数的函数
	UseCallback(DelayCode)

	// 调用 struct 的方法
	var user User
	fmt.Println("user is: ", user)
	user.NameChangedTo("new")
	fmt.Println("user.name is: ", user.GetName())

}

{% endhighlight %}

将上面的代码存入源文件 function.go 并使用 `go run function.go` 可以看到下面的输入：

{% highlight text %}
I am a function with no parameter and return nothing.
My parameters are:  1 2 OK
The double of 10 is:  20
Initial value of doubledInput of int type is:  0
The double of 10 is:  20
The Double of 10 is:  20
Test, The sum of paras is: 6
The 5 num of Fabinaci is: 5
Input is:  100
I will be executed before defer
Defer: I will be executed at first
Defer: I will be executed at last
I will be executed before defer
Defer: I will be executed at first
Defer: I will be executed at last
user is:  {}
user.name is:  new
{% endhighlight %}

注释中对函数的各种用法的解释很清楚，就不多说了。关于 `panic` 和 `recover` 在函数中的用法后面会专门讨论。

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

[1]: http://en.wikipedia.org/wiki/Associative_array
[2]: https://golang.org/ref/spec#Making_slices_maps_and_channels
[3]: http://golang.org/ref/spec#Semicolons
[4]: (http://coolshell.cn/articles/8460.html)


