---
layout: post
title: "Golang 语言基础之二： for, ifelse, switch"
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

## For 循环

Golang 中没有 `while` 语句。循环语句只有 `for`。而且配合 `range` 表达式可以方便地以迭代器的形式访问 Golang 中  `string`, `array`, `slice`, `map`, `channel` 类型对象的元素。

下面的实例代码演示了 `for` 语句的常用方法：

{% highlight go %}
package main

import "fmt"

func main() {
	v_str := "post hoc"
	fmt.Println("Test str: ", v_str)

	// 类似于 C/C++ 语言的传统使用方式
	for i := 0; i < 4; i++ {
		fmt.Printf("v_str[%d]: %q\t", i, v_str[i])
	}
	fmt.Println()

	// 同时对多个局部变量进行赋值
	for i, length := 0, len(v_str); i < length; i++ {
		fmt.Printf("v_str[%d]: %q\t", i, v_str[i])
	}
	fmt.Println()

	// 配合 range 使用
	// 下面代码中 elem 是 v_str 中每个元素值的副本，在循环体内可以对其进行任意修改，不会影响到原字符串
	for i, elem := range v_str {
		elem = elem - 32
		fmt.Printf("v_str[%d]: %q\t", i, elem)
	}
	fmt.Println("\nTest str: ", v_str)

	// 只保留条件判断：
	fmt.Println("Beginning 4 chars in v_str: ")
	j := 0
	for j < 4 {
		fmt.Printf("v_str[%d]: %q\t", j, v_str[j])
		j++
	}
	fmt.Println()

	// 无条件判断限制的循环，需要在合适的时候从内部 break 或者 return，否则无限循环
	count := 0
	for {
		if count >= 3 {
			break
		}
		fmt.Printf("count: %d\n", count+1)
		count++
	}

	v_IntArray := [...]int{1, 2, 3, 4, 5}

	// 注意：如果 range 处理的是一个指针的数组，那么在循环体内通过指针修改其指向的对象是可以的。
	// range 表达式处理数组，代码块中的改变不会影响到原始数组
	for i, elem := range v_IntArray {
		elem *= 100
		fmt.Println(i, elem)
	}
	fmt.Println("After range []T", v_IntArray)

	// range 表达式处理数组的指针，代码块中的改变也不会影响到原始数组
	for i, elem := range &v_IntArray {
		elem *= 100
		fmt.Println(i, elem)
	}
	fmt.Println("After range *[]T", v_IntArray)

	// range 表达式处理指针的数组，代码块中对指针指向内容的改变会改变原数组中指针指向的内容
	v_Pointer2IntArray := [...]*int{&v_IntArray[0], &v_IntArray[1], &v_IntArray[2], &v_IntArray[3], &v_IntArray[4]}
	for i, elem := range v_Pointer2IntArray {
		*elem *= 100
		fmt.Println(i, *elem)
	}
	fmt.Println("After range []*T, the values being pointed by pointers in v_Pointer2IntArray is:")
	for _, elem := range v_Pointer2IntArray {
		fmt.Printf("%d\t", *elem)
	}
}

{% endhighlight %}

将上面的代码存入源文件 for.go 并使用 `go run for.go` 可以看到下面的输入：

{% highlight text %}
Test str:  post hoc
v_str[0]: 'p'	v_str[1]: 'o'	v_str[2]: 's'	v_str[3]: 't'	
v_str[0]: 'p'	v_str[1]: 'o'	v_str[2]: 's'	v_str[3]: 't'	v_str[4]: ' '	v_str[5]: 'h'	v_str[6]: 'o'	v_str[7]: 'c'	
v_str[0]: 'P'	v_str[1]: 'O'	v_str[2]: 'S'	v_str[3]: 'T'	v_str[4]: '\x00'	v_str[5]: 'H'	v_str[6]: 'O'	v_str[7]: 'C'	
Test str:  post hoc
Beginning 4 chars in v_str: 
v_str[0]: 'p'	v_str[1]: 'o'	v_str[2]: 's'	v_str[3]: 't'	
count: 1
count: 2
count: 3
0 100
1 200
2 300
3 400
4 500
After range []T [1 2 3 4 5]
0 100
1 200
2 300
3 400
4 500
After range *[]T [1 2 3 4 5]
0 100
1 200
2 300
3 400
4 500
After range []*T, the values being pointed by pointers in v_Pointer2IntArray is:
100	200	300	400	500
{% endhighlight %}

需要注意的是：

- `for` 可以和 `range` 配合，在使用中类似于 C# 中的 `foreach` 或者 Python 中的 `with`。
- `for` 配合 `range` 使用时，会对数组元素进行值拷贝。如果 `range` 处理的是一个指针的数组，那么在循环体内通过指针修改其指向的对象是可以的。
- 关于 `range` 的详细讨论在 [之四： map，range][2]

## IF-ELSE 语句

比较简单，看下面的实例就可以了：

{% highlight go %}
package main

import "fmt"

func main() {
	v_int := 9

	if v_int < 10 {
		fmt.Println("true")
	}

	// 可以省略条件判断的小括号
	if v_int < 10 {
		fmt.Println("true")
	}

	// 可以定义代码块作用域的局部变量
	if temp := 9; v_int > 0 {
		fmt.Printf("%d / %d = %d", temp, v_int, temp/v_int)
	}

	// 在进行条件判断前，可以执行一些初始化的操作，比如打印一些信息
	// else 必须跟在上一个代码块花括号右半部分后面，否则会有编译错误
	if fmt.Println("Checking number:"); v_int%2 == 0 {
		fmt.Printf("%d is even number\n", v_int)
	} else {
		fmt.Printf("%d is odd number\n", v_int)
	}
}

{% endhighlight %}

将上面的代码存入源文件 ifelse.go 并使用 `go run ifelse.go` 可以看到下面的输入：

{% highlight text %}
true
true
9 / 9 = 1Checking number:
9 is odd number
{% endhighlight %}

需要注意的地方：

- 在条件判断语句前可以加入初始化语句。
- `else` 必须跟在上个代码块结束的右半部分花括号后面，否则编译出错。，因为 Golang 编译器会在 `{` 前面自动加 `;`。
- 和 `for` 语句一样，代码块的左半部分花括号必须在条件判断语句同一行，因为 Golang 编译器会在 `{` 前面自动加 `;`。

## Switch 语句

{% highlight go %}
package main

import "fmt"

func main() {
	// 可以替代 if-else 语句
	v_int := 73
	switch {
	case v_int < 100:
		fmt.Println(v_int, "is less than 100")
	case v_int >= 100:
		fmt.Println(v_int, "is greater than or enqual to 100")
	}

	// 分支可以使用变量
	v_intArr := []int{1, 2, 3}
	v_int = 2
	switch v_int {
	case v_intArr[1]:
		println("hello")
	case len(v_intArr):
		println("b")
	default:
		println("c")
	}
	
	// 使用 fallthrough 之后从执行下一个分支，而且不判断条件。注意 20 是偶数
	v_int = 20
	switch v_int {
	case 20:
		println(v_int, "is even, but:")
		fallthrough
	case 0:
		fmt.Println(v_int, "is odd number")
	}
}

{% endhighlight %}

将上面的代码存入源文件 switch.go 并使用 `go run switch.go` 可以看到下面的输入：

{% highlight text %}
73 is less than 100
20 is odd number

hello
20 is even, but:
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

[1]: http://golang.org/ref/spec#For_statements
[2]: http://xhrwang.me/2014/12/22/golang-fundamentals-4-map-range.html



