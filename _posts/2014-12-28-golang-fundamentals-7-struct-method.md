---
layout: post
title: "Golang 语言基础之七： struct, method"
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

很多现代语言都支持面向对象编程模式，C++，Python，Ruby，C#，F# 等，其中 Python，Ruby，F# 甚至加入了很多对 [函数式编程][1] 的支持。Golang 作为一门相对来说很年轻的语言，通过对闭包的支持，也可以用于实践函数式编程模式，这有便于我们写出抽象程度很高的代码，而且语言本身的灵活性很高。

至于对面向对象编程模式的支持，Golang 中没有 `class` 的概念，虽然有 `struct` 和 `interface` 类型，但是不支持像 C/C++ 语言中传统方式的 `继承`，实际上 Golang 对 `OOP 特性` 的直接支持就只有 `封装`。我们可以使用 `struct` 通过一些语法来模拟 `继承关系`。下面关于 `struct` 的用法实例中会给出具体的例子。而 `多态`，虽然 Golang 是静态类型的语言，但是 `struct` 和 `interface` 配合 Golang 中 `method` 的概念，使用时却很类似于动态语言的 [鸭子类型][2] 对 `多态` 的实现， Golang 通过 `struct` 和 `interface` 的方法集及其规则来保证可行性。

官方文档的 FAQ 中有一个问题：[Is Go an object-oriented language?][3]，Google 是这么回答的：

> Yes and no. Although Go has types and methods and allows an object-oriented style of programming, there is no type hierarchy. The concept of “interface” in Go provides a different approach that we believe is easy to use and in some ways more general. There are also ways to embed types in other types to provide something analogous—but not identical—to subclassing. Moreover, methods in Go are more general than in C++ or Java: they can be defined for any sort of data, even built-in types such as plain, “unboxed” integers. They are not restricted to structs (classes).


## Struct, Method 和 Method Sets

先看看 [官方定义][4]

> A struct is a sequence of named elements, called fields, each of which has a name and a type. Field names may be specified explicitly (IdentifierList) or implicitly (AnonymousField). Within a struct, non-blank field names must be unique.

也就是说：一个 `struct` 类型由一系列被称为 `属性` 的命名元素组成。每个 `属性` 由 `属性名` 和 `属性类型` 组成。`属性名` 可以被显式地指定，也可以是匿名的。`struct` 类型中 `非补位` 的所有属性（属性名非下划线 `_`）的 `属性名` 不能重复。

__匿名属性__：`匿名属性`是只包含 属性类型 的 `属性`,也被称为 `嵌入属性` 或者 `一个类型在这个 struct 中的嵌入`。这里的 `类型` 可以是指针类型，但不能是指针的指针，而且一个 `struct` 类型不能同时包含一个类型本身以及指向该类型的指针，因为他们在这个 `struct` 类型中的 `属性名` 是相同的，都是不包含 `package name` 的类型名。看一下[官网的说明][4]：

{% highlight text %}
// A struct with four anonymous fields of type T1, *T2, P.T3 and *P.T4
struct {
	T1        // field name is T1
	*T2       // field name is T2
	P.T3      // field name is T3
	*P.T4     // field name is T4
	x, y int  // field names are x and y
}
{% endhighlight %}

一个 `struct` 类型的 `匿名属性` 的方法可以通过该 `struct` 对象直接调用。这个特性可以让我们实现类似于传统 OOP 中的 `继承` 特性。调用方法时 Golang 会从外层向内层寻找，所以如果想 `override` 一个 `匿名属性` 的方法，可以给这个 `struct` 类型定义一个具有和其具有相同函数签名的方法，这样调用时 Golang 在最外层就能找到符合条件的方法并调用。

__方法集__：对于一个 `struct` 类型 S 和 一个类型 T 而言，如果 T 会作为 S 的一个 `匿名属性`，S 的方法集（[method sets][5]）遵循下面的[规则][4]：

> - If S contains an anonymous field T, the method sets of S and *S both include promoted methods with receiver T. The method set of *S also includes promoted methods with receiver *T.  
> - If S contains an anonymous field *T, the method sets of S and *S both include promoted methods with receiver T or *T.

我们还是用用法实例来理解一下，使用实例如下：

{% highlight go %}
package main

import "fmt"

// 定义空 `struct` 类型
type EmptyStruct struct{}

// 定义含有 `tag` 的 `struct` 类型，之后可以用 reflection 对有 `tag` 的属性进行访问。
type Job struct {
	title string "JD"
	rank  int
}

// 定义不含有 `tag`、其他属性和 Job `struct` 类型完全相同的另一个 `struct` 类型
type JobWithoutTag struct {
	title string
	rank  int
}

// 定义 `struct` 类型 Job 的方法，由于方法体中无需用到 Job 对象，那么对象名可以省略
func (Job) Help() {
	fmt.Println("Use \"j.What()\" to get job detail.")
}

// 定义 `struct` 类型 Job 的方法，由于方法体中用到 Job 对象，那么对象名不可以省略
func (job Job) What() {
	fmt.Printf("Job Detail:\n\ttitle: %s\n\trank: %d\n", job.title, job.rank)
}

// 定义指向 `struct` 类型 Job 的指针的方法
func (jobP *Job) SetRank(newRank int) {
	jobP.rank = newRank
	fmt.Println("Rank updated, new value is: ", jobP.rank)
}

// 定义 Employee 结构类型，其含有 `补位属性`、匿名 `struct` 类型属性、匿名 `struct` 类型而显式命名的属性
// 同一类型的属性可以放在一行
// 从效果看，Employee `继承` 了 Job，这和传统 `继承` 区别很大，注意上面提到的 `匿名属性` 的方法集规则
// 在其他 OOP 语言比如 C++ 中，我们不会做这样的抽象设计，因为两者没有什么所谓的 `继承关系`
type Employee struct {
	_                   int
	age                 int
	firstName, lastName string
	pack                struct {
		salary int
		stock  int
	}
	*Job
}

// 定义 `struct` 类型 Employee 的方法，由于其匿名属性 Job 也有同名方法，根据上面提到的规则
// Employee 的 Help 方法会 override 其匿名属性 Job 的同名方法。
func (Employee) Help() {
	fmt.Println("Use \"e.Job.What()\" to get job detail of current employee.")
}

// 定义 `struct` 类型，这个类型唯一的属性为空接口类型，而任何类型都实现了空接口
// 所以 OnlyInterface 可以实例化属性为任何类型的 `struct` 对象
type OnlyInterface struct {
	f interface{}
}

func main() {

	// 检查空 `struct` 类型
	varEmptyStruct := EmptyStruct{}
	fmt.Println("varEmptyStruct: ", varEmptyStruct)

	// 检查 `struct` 类型对象的默认值
	var varJob0 Job
	fmt.Println("varJob0 = ", varJob0)

	// 对 `struct` 类型对象进行顺序初始化
	varJob1 := Job{
		"CEO",
		99,
	}
	fmt.Println("varJob1 = ", varJob1)

	// 对 `struct` 类型对象按照属性名依次初始化，可以和属性定义的顺序不一致
	varJob2 := Job{
		rank:  99,
		title: "CEO",
	}
	fmt.Println("varJob2 = ", varJob2)

	// 使用属性名进行初始化可以选择性初始化个别属性，其他属性初始化为它们的默认值
	varJob3 := Job{
		title: "COO",
	}
	fmt.Println("varJob2 = ", varJob3)

	// Employee  `struct` 类型对象的默认值
	varEmployee0 := Employee{}
	fmt.Println("varEmployee = ", varEmployee0)

	// 初始化 Employee `struct` 类型对象，匿名属性需要在初始化其他属性后使用对象名显式赋值
	varEmployee1 := Employee{
		age:       50,
		firstName: "Jack",
		lastName:  "Ma",
	}
	varEmployee1.pack.salary = 100000000
	varEmployee1.pack.stock = 1000000
	varJob4 := Job{
		"Founder",
		100,
	}
	varEmployee1.Job = &varJob4
	fmt.Println("varEmployee = ", varEmployee1)
	fmt.Println("varEmployee.Job = ", *(varEmployee1.Job))

	// 注意 Employee 的 Help 方法 `override` 了其属性 Job 的 Help 方法
	varEmployee1.Help()
	// 调用 Job 的 Help 方法需要显式调用
	varEmployee1.Job.Help()
	// 使用 Employee 对象可以直接调用其属性 Job 的方法 What
	varEmployee1.What()
	// Employee 对象的属性 Job 的类型是 `*Job`，所以可以修改 Job 的内容
	// 如果其类型是 Job 本身，由于获取到的 `struct` 对象是其原始值的拷贝，修改不会生效
	varEmployee1.Job.SetRank(99)
	fmt.Println("After job rank change, varEmployee.Job = ", *(varEmployee1.Job))

	// `struct` 类型对象是值类型，可以进行比较运算
	varEmployee2 := Employee{
		age:       50,
		firstName: "Jack",
		lastName:  "Ma",
	}
	varEmployee2.pack.salary = 100000000
	varEmployee2.pack.stock = 1000000
	varEmployee2.Job = &varJob4
	if fmt.Println("Cmpare 2 struct object."); varEmployee2 == varEmployee1 {
		fmt.Println("varEmployee2 equals varEmployee1.")
	}

	// Tag 是 `struct` 类型的一部分，下面 varJob5 无法赋值给 varJob4
	// varJob5 := JobWithoutTag{
	// 	"Founder",
	// 	100,
	// }
	// varJob4 = varJob5

	// 属性类型为空接口的 `struct` 类型的使用
	varOnlyInterface := OnlyInterface{
		f: 100,
	}
	fmt.Println("OnlyInterface with int: ", varOnlyInterface)
	varOnlyInterface = OnlyInterface{
		f: "I like Golang.",
	}
	fmt.Println("OnlyInterface with string: ", varOnlyInterface)

}

{% endhighlight %}

将上面的代码存入源文件 struct.go 并使用 `go run struct.go` 可以看到下面的输入：

{% highlight text %}
varEmptyStruct:  {}
varJob0 =  { 0}
varJob1 =  {CEO 99}
varJob2 =  {CEO 99}
varJob2 =  {COO 0}
varEmployee =  {0 0   {0 0} <nil>}
varEmployee =  {0 50 Jack Ma {100000000 1000000} 0xc20801e0c0}
varEmployee.Job =  {Founder 100}
Use "e.Job.What()" to get job detail of current employee.
Use "j.What()" to get job detail.
Job Detail:
	title: Founder
	rank: 100
Rank updated, new value is:  99
After job rank change, varEmployee.Job =  {Founder 99}
Cmpare 2 struct object.
varEmployee2 equals varEmployee1.
OnlyInterface with int:  {100}
OnlyInterface with string:  {I like Golang.}
{% endhighlight %}


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

[1]: http://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B8%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80
[2]: http://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B
[3]: http://golang.org/doc/faq#Is_Go_an_object-oriented_language
[4]: http://golang.org/ref/spec#Struct_types
[5]: http://golang.org/ref/spec#Method_sets


