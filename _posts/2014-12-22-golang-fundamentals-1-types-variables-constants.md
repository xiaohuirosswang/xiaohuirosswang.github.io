---
layout: post
title: "Golang 语言基础之一： type, variable, constant"
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


最近开始系统地学习 Golang，之前走马观花地看过 [酷壳上陈皓的介绍][8]，从一名有 C++、 C# 开发经验的程序员角度看，还是有些和思维惯性不一致的地方，后来使用 Python 开发过一段时间之后，再看 Golang，就感觉亲切多了。

Golang 本身可以使用 `go run` 以解释型语言的方式运行（实际上经过了编译），Apple 最新推出的 Swift 在这个方面和 Golang 有点不约而同的类似。新编程语言的设计者都是业界大牛，我们可以感觉到他们在设计一门新语言的时候，一方面吸收已有编程语言的优点，明确新语言的开发领域，另一方面都不同程度支持函数型编程范式，也都自带 GC，保证开发效率。所以在我们根据自己的兴趣学习这些语言的时候，是可以一定程度上触类旁通的，语言本身不同的地方我个人觉得和语言设计者对编程语言发展的理解以及语言本身针对的开发领域不同关系更多点。


> Go is expressive, concise, clean, and efficient. Its concurrency mechanisms make it easy to write programs that get the most out of multicore and networked machines, while its novel type system enables flexible and modular program construction. Go compiles quickly to machine code yet has the convenience of garbage collection and the power of run-time reflection. It's a fast, statically typed, compiled language that feels like a dynamically typed, interpreted language.

------------------

这是 [Google 对 Golang 语言的官方介绍][1]。OK, Go，gophers！

Golang 语言基础系列是我个人学习 Golang 的笔记，以 [Go by Example][9] 的用实例学 Golang 为结构参考。

## 类型

### 基本类型

[Go 学习笔记这本书][2] 中对 Golang 的类型做了一个总结：

![Golang Types][3]

我写了一段 Golang 代码（[Gist][4]）来检查 Numeric 类型的取值范围：

{% highlight go %}
package main

import (
	"fmt"
	"math"
	"strconv"
	"unsafe"
)

func main() {
	var v_bool bool
	var v_byte byte
	var v_rune rune
	var v_int8 int8
	var v_int16 int16
	var v_int32 int32
	var v_int64 int64
	var v_uint8 uint8
	var v_uint16 uint16
	var v_uint32 uint32
	var v_uint64 uint64
	var v_float32 float32
	var v_float64 float64
	fmt.Printf("%-20s%-20s%-20s%-50s\n",
		"Type", "Sizeof", "DefaultValue", "Description")
	fmt.Printf("%-20s%-20d%-20t%-50s\n",
		"bool", unsafe.Sizeof(v_bool), v_bool, "")
	fmt.Printf("%-20s%-20d%-20d%-50s\n",
		"byte", unsafe.Sizeof(v_byte), v_byte, "0 - "+strconv.Itoa(math.MaxUint8))
	fmt.Printf("%-20s%-20d%-20d%-50s\n",
		"rune", unsafe.Sizeof(v_rune), v_rune, strconv.Itoa(math.MinInt32)+" - "+strconv.Itoa(math.MaxInt32))
	fmt.Printf("%-20s%-20d%-20d%-50s\n",
		"int8", unsafe.Sizeof(v_int8), v_int8, strconv.Itoa(math.MinInt8)+" - "+strconv.Itoa(math.MaxInt8))
	fmt.Printf("%-20s%-20d%-20d%-50s\n",
		"int16", unsafe.Sizeof(v_int16), v_int16, strconv.Itoa(math.MinInt16)+" - "+strconv.Itoa(math.MaxInt16))
	fmt.Printf("%-20s%-20d%-20d%-50s\n",
		"int32", unsafe.Sizeof(v_int32), v_int32, strconv.Itoa(math.MinInt32)+" - "+strconv.Itoa(math.MaxInt32))
	fmt.Printf("%-20s%-20d%-20d%-50s\n",
		"int64", unsafe.Sizeof(v_int64), v_int64, strconv.Itoa(math.MinInt64)+" - "+strconv.Itoa(math.MaxInt64))
	fmt.Printf("%-20s%-20d%-20d%-50s\n",
		"uint8", unsafe.Sizeof(v_uint8), v_uint8, "0 - "+strconv.Itoa(math.MaxUint8))
	fmt.Printf("%-20s%-20d%-20d%-50s\n",
		"uint16", unsafe.Sizeof(v_uint16), v_uint16, "0 - "+strconv.Itoa(math.MaxUint16))
	fmt.Printf("%-20s%-20d%-20d%-50s\n",
		"uint32", unsafe.Sizeof(v_uint32), v_uint32, "0 - "+strconv.Itoa(math.MaxUint32))
	fmt.Printf("%-20s%-20d%-20d%-50s\n",
		"uint64", unsafe.Sizeof(v_uint64), v_uint64, "0 - "+strconv.FormatUint(math.MaxUint64, 10))
	fmt.Printf("%-20s%-20d%-20f%-50s\n",
		"float32", unsafe.Sizeof(v_float32), v_float32, "Max: "+strconv.FormatFloat(math.MaxFloat32, 'f', 6, 32))
	fmt.Printf("%-20s%-20d%-20f%-50s\n",
		"float64", unsafe.Sizeof(v_float64), v_float64, "")
}
{% endhighlight %}

将上面的代码存入源文件 types.go 并使用 `go run types.go` 可以看到下面的输入：

{% highlight text %}
Type                Sizeof              DefaultValue        Description                                       
bool                1                   false                                                                 
byte                1                   0                   0 - 255                                           
rune                4                   0                   -2147483648 - 2147483647                          
int8                1                   0                   -128 - 127                                        
int16               2                   0                   -32768 - 32767                                    
int32               4                   0                   -2147483648 - 2147483647                          
int64               8                   0                   -9223372036854775808 - 9223372036854775807        
uint8               1                   0                   0 - 255                                           
uint16              2                   0                   0 - 65535                                         
uint32              4                   0                   0 - 4294967295                                    
uint64              8                   0                   0 - 18446744073709551615                          
float32             4                   0.000000            Max: 340282346638528859811704183484516925440.000000
float64             8                   0.000000
{% endhighlight %}

这里需要注意的是：

- 变量被声明后会自动初始化为类型的默认值，如果初始化的时候给了初始值则使用该值。
- 对于引用类型来说，空值为 `nil`。
- `uintptr` 能够用来存储 32bit 和 64bit 的地址。
- `uint` 可以表示 32bit 或者 64bit 的整型数。
- `int` 是 `uint` 的别名。
- `rune` 是 `int32` 的别名，可以用来存储 Unicode 标准中定义的任一字符。
- `byte` 是 `uint8` 的别名，
- 在表达式中进行计算时如果涉及多个不同的数字类型，类型转换必须显式定义，以保证可移植性：

	> To avoid portability issues all numeric types are distinct except byte, which is an alias for uint8, and rune, which is an alias for int32. Conversions are required when different numeric types are mixed in an expression or assignment. For instance, int32 and int are not the same type even though they may have the same size on a particular architecture.

### 引用类型

和上面图片中总结的一样，Golang 中的引用类型有 `slice`，`map` 和 `channel`。关于这三种复杂类型的详细讨论会有专门的内容，这里以 slice 为例，看看引用类型的初始化和使用：

{% highlight go %}
package main

import "fmt"

func main() {

	// 通过初始化表达式明确定义 slice 里面每个元素
	v_slice := []int{0, 0, 0}
	fmt.Println("v_slice: ", v_slice)

	// 修改 slice 中的元素值
	v_slice[1] = 10
	fmt.Println("v_slice: ", v_slice)

	// 通过内置函数 make 初始化 slice。最后一个参数表示 Capacity，如果省略，则默认值等于第二个参数 length。
	// 编译时编译器会将 make 翻译为 makeslice 进行对象创建并返回对象。
	v_slice = make([]int, 3, 3)
	v_slice[2] = 10
	fmt.Println("v_slice: ", v_slice)

	// 通过内置函数 new 初始化 slice，省略了 Capacity。编译器创建对象后返回对象指针
	// 下面的官方文档中说这种用法基本没什么用
	// https://golang.org/doc/effective_go.html#slices
	vp_slice := new([]int)
	fmt.Println("vp_slice: ", vp_slice)
	fmt.Println("vp_slice len: ", len(*vp_slice))
	fmt.Println("vp_slice capacity: ", cap(*vp_slice))
}

{% endhighlight %}

将上面的代码存入源文件 refer.go 并使用 `go run refer.go` 可以看到下面的输入：

{% highlight text %}
v_slice:  [0 0 0]
v_slice:  [0 10 0]
v_slice:  [0 0 10]
vp_slice:  &[]
vp_slice len:  0
vp_slice capacity:  0
{% endhighlight %}

类型中的字符串 `string` 和指针 `pointer` 在 [Golang 语言基础之六： string, pointer](/2014/12/27/golang-fundamentals-6-string-pointer.html) 中有详细介绍。

类型学习资源：

- [官方文档中关于 Types 的部分][5]
- [rune 类型的解释][6]
- [Go 学习笔记][2] 中的类型相关章节

## 变量

Golang 中变量的声明和 C/C++ 有很大的不同，先看一段代码：

{% highlight go %}
package main

import "fmt"

var (
	g_str  string = "TBB"
	g_Int  int    = 73
	g_Bool bool   = true
)

func main() {

	// 使用 var 关键字声明一个变量，变量类型跟在变量名后面
	var a string = "Golang"
	fmt.Println(a)

	// 声明变量时如果赋予了初始值，则可以省略类型，编译器会自动根据给定的初始化值进行推断
	var d = true
	fmt.Println(d)

	// 使用 var 也可以同时声明多个变量，Golang 会对变量从左至右依次赋值
	// 多个变量属于不同类型时
	var v_int, int_value = "v_int is: ", 2
	fmt.Println(v_int, int_value)
	// 多个变量属于相同类型时
	var b, c int = 1, 2
	fmt.Println(b, c)

	// 如果声明变量的时候没有给定初始化值，变量会被初始化为其类型的空值
	var e int
	fmt.Println(e)

	// 在函数内部声明局部变量时，:= 语法可以用来简化已有初始化值的变量声明，下面的语句和 var f string = "short" 是等价的
	// 使用这种用法的时候需要很小心，如果不小心忘掉了 ：符号，就有可能变成对同名的全局变量赋值了。
	f := "short term"
	fmt.Println(f)

	// 声明一个空的 slice 变量并打印
	v_slice := make([]string, 3)
	fmt.Println("Empty slice: ", v_slice)
	v_slice = nil
	fmt.Println("nil slice: ", v_slice)

	fmt.Println(g_Int, g_Bool, g_str)
}

{% endhighlight %}

将上面的代码存入源文件 varialbles.go 并使用 `go run varialbles.go` 可以看到下面的输入：

{% highlight text %}
Golang
true
v_int is:  2
1 2
0
short term
Empty slice:  [  ]
nil slice:  []
73 true TBB
{% endhighlight %}

注意：

- 全局变量，也就是声明在函数体外部的变量可以在当前包以及引用了当前包的代码中访问到。
- 局部变量，声明在函数体内的变量，作用域只在当前函数。for，switch，if-else 块内部的变量的作用域在当前块中，可以和外部变量同名，同名时相当于在当前代码块隐藏了外部变量。
- 未使用的局部变量会引发编译错误。
 
更详细的变量说明可以参考：

- [The Way to Go][7] 中关于变量的章节


## 常量

常量的声明方式和变量有些不同的地方：

{% highlight go %}
package main

import (
	"fmt"
	"unsafe"
)

// 使用 const 关键字同时声明多个常量
const int_x, int_y int = 1, 2

// 常量也可以省略类型名让编译器进行类型推断
const str = "Golang!"

// 通过常量数组声明多个常量，如果不同的常量有相同的值，可以省略类型名和初始值
const (
	a, b = 1001001, 73
	c    = true
	d
)

func main() {
	fmt.Println(int_x, int_y)
	fmt.Println(str)
	fmt.Println(a, b, c, d)

	// 常量的值可以是编译期可以确定值的函数比如 len，unsafe.Sizeof 等的返回值
	const sizeofInt = unsafe.Sizeof(int_x)
	fmt.Printf("sizeofInt: %d", sizeofInt)

	// 和变量不同，未使用的局部常量不会导致编译错误
	const no_use = false

}
{% endhighlight %}

将上面的代码存入源文件 constants.go 并使用 `go run constants.go` 可以看到下面的输入：

{% highlight text %}
1 2
Golang!
1001001 73 true true
sizeofInt: 8
{% endhighlight %}

## 参考资料

- [The Go Programming Language](http://golang.org/cmd/go/)
- [Go in Action  中文版](https://github.com/astaxie/Go-in-Action)
- [The way to Go 中文版](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/02.2.md)
- [Go by Example](https://gobyexample.com/hello-world)
- [Organizing Go code](https://talks.golang.org/2014/organizeio.slide#1)
- [Testing Techniques](https://talks.golang.org/2014/testing.slide#1)
- [Go 语言分享](http://www.jiagoushi.me/index.php/archives/43/)
- [Go 学习笔记](https://github.com/qyuhen/book)
- [Go 语言简介](http://coolshell.cn/articles/8460.html)
- [Tony Bai 的博客](http://tonybai.com/)


### 本文内容包括 Golang 的 类型、变量以及常量。


[1]: http://golang.org/doc/
[2]: https://github.com/qyuhen/book
[3]: /images/go_types.png
[4]: https://gist.github.com/xhrwang/3a4d879898ced496b971
[5]: http://golang.org/ref/spec#Types
[6]: http://stackoverflow.com/questions/17855774/go-rune-type-explanation/17856102#17856102
[7]: https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/04.4.md
[8]: http://coolshell.cn/articles/8460.html
[9]: https://gobyexample.com/


