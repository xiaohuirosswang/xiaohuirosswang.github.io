---
layout: post
title: "Golang 语言基础之八： interface"
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

我们知道 Golang 中没有 `class` 的概念，而是通过 `interface` 类型转换支持在动态类型语言中常见的 [鸭子类型][3] 达到运行时多态的效果。

[官方文档][1] 中对 `Interface` 是这样定义的：

> An interface type specifies a method set called its interface. A variable of interface type can store a value of any type with a method set that is any superset of the interface. Such a type is said to implement the interface. The value of an uninitialized variable of interface type is nil.

一个 `interface` 类型定义了一个 [方法集][2] 作为其 `接口`。 `interface` 类型的变量可以保存含有属于这个 `interface` 类型方法集超集的任何类型的值，这时我们就说这个类型 `实现` 了这个 `接口`。未被初始化的 `interface` 类型变量的零值为 `nil`。

对于 `interface` 类型的方法集来说，其中每一个方法都必须有一个不重复并且不是 `补位名`（即单下划线 `_`）的方法名。

拿 [官方文档的例子][1] 来说:

如果我们定义 `File` 接口类型：

{% highlight go %}
// A simple File interface
type File interface {
	Read(b Buffer) bool
	Write(b Buffer) bool
	Close()
}
{% endhighlight %}

那么，假设有两个其他类型 `S1` 和 `S2` 都拥有下面的方法集：

{% highlight go %}
func (p T) Read(b Buffer) bool { return … }
func (p T) Write(b Buffer) bool { return … }
func (p T) Close() { … }
{% endhighlight %}

注意，上面代码中的 `T` 代表 `S1` 或者 `S2`。这时，无论 `S1` 和 `S2` 各自的方法集中还有哪些方法，我们都说 `S1` 和 `S2` 各自 `实现` 了接口 `File`。

对于空接口 `interface {}` 来说，由于其不含有任何方法签名，在 Golang 语言中我们可以认为所有类型都 `实现` 了空接口。

一个类型可以 `实现` 一个或者多个接口，假设我们还有下面的接口类型：

{% highlight go %}
type Locker interface {
	Lock()
	Unlock()
}
{% endhighlight %}

而 `S1` 和 `S2` 各自的方法集中都含有下面的方法签名：

{% highlight go %}
func (p T) Lock() { … }
func (p T) Unlock() { … }
{% endhighlight %}

同样的，上面代码中的 `T` 代表 `S1` 或者 `S2`，这时我们称 `S1` 和 `S2` 各自都 `实现` 了接口 `File` 和 `Locker`。

一个接口类型（假设为 `T`）可以 `嵌入` 另一个接口类型(假设为 `E`)，这被称为 `embedding interface E in T`，而且，所有 `E` 中的方法都会被添加到 `T` 中。需要注意的是，一个接口不能 `嵌入` 其本身，也不能 `嵌入` 任何已经 `嵌入` 其本身的其他接口，这样会造成递归 `嵌入`。

还是通过下面的例子来看看：

{% highlight go %}
package main

import "fmt"

// 定义接口类型 PeopleGetter 包含获取基本信息的方法
type PeopleGetter interface {
	GetName() string
	GetAge() int
}

// 定义接口类型 EmployeeGetter 包含获取薪水的方法
// EmployeeGetter 接口中嵌入了 PeopleGetter 接口，前者将获取后者的所有方法
type EmployeeGetter interface {
	PeopleGetter
	GetSalary() int
	Help()
}

// 定义结构 Employee
type Employee struct {
	name   string
	age    int
	salary int
	gender string
}

// 定义结构 Employee 的方法
func (self *Employee) GetName() string {
	return self.name
}

func (self *Employee) GetAge() int {
	return self.age
}

func (self *Employee) GetSalary() int {
	return self.salary
}

func (self *Employee) Help() {
	fmt.Println("This is help info.")
}

// 匿名接口可以被用作变量或者结构属性类型
type Man struct {
	gender interface {
		GetGender() string
	}
}

func (self *Employee) GetGender() string {
	return self.gender
}

// 定义执行回调函数的接口
type Callbacker interface {
	Execute()
}

// 定义函数类型 func() 的新类型 CallbackFunc
type CallbackFunc func()

// 实现 CallbackFunc 的 Execute() 方法
func (self CallbackFunc) Execute() { self() }

func main() {
	// 空接口的使用，空接口类型的变量可以保存任何类型的值
	// 空格口类型的变量非常类似于弱类型语言中的变量
	var varEmptyInterface interface{}
	fmt.Printf("varEmptyInterface is of type %T\n", varEmptyInterface)
	varEmptyInterface = 100
	fmt.Printf("varEmptyInterface is of type %T\n", varEmptyInterface)
	varEmptyInterface = "Golang"
	fmt.Printf("varEmptyInterface is of type %T\n", varEmptyInterface)

	// Employee 实现了 PeopleGetter 和 EmployeeGetter 两个接口
	varEmployee := Employee{
		name:   "Jack Ma",
		age:    50,
		salary: 100000000,
		gender: "Male",
	}
	fmt.Println("varEmployee is: ", varEmployee)
	varEmployee.Help()
	fmt.Println("varEmployee.name = ", varEmployee.GetName())
	fmt.Println("varEmployee.age = ", varEmployee.GetAge())
	fmt.Println("varEmployee.salary = ", varEmployee.GetSalary())

	// 匿名接口对象的使用
	varMan := Man{&Employee{
		name:   "Nobody",
		age:    20,
		salary: 10000,
		gender: "Unknown",
	}}
	fmt.Println("The gender of Nobody is: ", varMan.gender.GetGender())

	// 接口类型转换，从超集到子集的转换是可以的
	// 从方法集的子集到超集的转换会导致编译错误
	// 这种情况下 switch 不支持 fallthrough
	var varEmpInter EmployeeGetter = &varEmployee
	switch varEmpInter.(type) {
	case nil:
		fmt.Println("nil")
	case PeopleGetter:
		fmt.Println("PeopleGetter")
	default:
		fmt.Println("Unknown")
	}

	// 使用 “执行回调函数的接口对象” 执行回调函数
	// 这种做法的优势是函数显式地 “实现” 特定接口
	varCallbacker := CallbackFunc(func() { println("I am a callback function.") })
	varCallbacker.Execute()

}
{% endhighlight %}

将上面的代码存入源文件 interface.go 并使用 `go run interface.go` 可以看到下面的输入：

{% highlight text %}
varEmptyInterface is of type <nil>
varEmptyInterface is of type int
varEmptyInterface is of type string
varEmployee is:  {Jack Ma 50 100000000 Male}
This is help info.
varEmployee.name =  Jack Ma
varEmployee.age =  50
varEmployee.salary =  100000000
The gender of Nobody is:  Unknown
PeopleGetter

I am a callback function.
{% endhighlight %}

关于 `interface` 更多需要注意的地方还有：

- 接口类型不包含任何数据成员。
- 接口类型进行转换的时候，默认返回的是临时对象，所以如果需要对原始对象进行修改，需要获取原始对象的指针。

这里有一篇关于 `interface` 的很好的文章： [How to use interfaces in Go][4]

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

[1]: http://golang.org/ref/spec#Interface_types
[2]: http://golang.org/ref/spec#Method_sets
[3]: http://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B
[4]: http://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go


