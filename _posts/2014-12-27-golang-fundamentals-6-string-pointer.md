---
layout: post
title: "Golang 语言基础之六： string, pointer"
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

## 字符串类型 `string`

字符串在 Golang 源码文件 `runtime.h` 中的定义如下：

{% highlight c %}
struct String
{
byte* str;
intgo len;
};
{% endhighlight %}

可以看出，其内部包含一个 `byte` 类型的数组，Golang 采用 UTF-8 编码方式。和 C/C++ 的字符数组不同，Golang 的字符数组有下面的特点：

- `string` 类型的零值为空字符串。
- 不能用序号获取字节元素指针，&s[i] 为非法。
- `string` 是不可变类型，其内部存储数据的字符数组无法修改。如需修改，需要将 `string` 对象转化为 `rune` 类型（如果字符串含有非 ASIC 码字符）或者 `byte` 类型（如果字符串由 ASIC 码字符组成）的 `slice` 对象。修改后的字符数组会重新分配内存保存在一个新的字符串对象中。
- 字节数组尾部不包含 `\0`

关于 `string` 的使用方法我们看个例子：

{% highlight go %}
package main

import "fmt"

func main() {

	// 默认 string 类型对象零值为空字符串，尾部不包含 `\0`
	var emptyStr string
	fmt.Println("emptyStr is: ", emptyStr)
	fmt.Println("len(emptyStr) is: ", len(emptyStr))

	// 声明 string 对象并初始化，使用下标访问。
	// 注意，如果字符串中包含中文等非 ASIC 码的字符，则使用下标索引会导致与期望不符的结果
	// 字符串是 UTF-8 编码，所以非 ASIC 码字符不止一个字节，而使用下标获得的是每个字节的内容。
	str := "I like 高圆圆"
	fmt.Println("string object is: ", str)
	fmt.Println("len(str) = : ", len(str))
	fmt.Println("str[1]: ", str[1])

	// 使用 ` 语法可以声明无转义的字符串
	str = `I
		Love
		Golang`

	fmt.Println("str: ", str)

	// 修改字符串，注意将字符串分别转化为 `rune` 和 `byte` 的 `slice` 对象。
	// 它们的长度是不相等的，`rune` 切片对象的长度等于原始 `string` 对象中的字符个数。
	// `byte` 切片对象的长度等于原始 `string` 对象在内存中所占的字节数，其值和 `len` 取得的 `string` 对象长度相等
	// 如果 `string` 对象中有非 ASIC 码，则两者的长度是不相等的。
	// 所以为了保证兼容性，最好将 `string` 对象转化为 `rune` 切片对象后进行修改。
	str = "I like 高圆圆"
	fmt.Println("Before modification, str: ", str)
	fmt.Println("len(str) = : ", len(str))
	rune_str := []rune(str)
	byte_str := []byte(str)
	fmt.Println("len([]rune(str)) = ", len(rune_str))
	fmt.Println("len([]byte(str)) = ", len(byte_str))
	rune_str[7] = '范'
	rune_str[8] = '冰'
	rune_str[9] = '冰'
	fmt.Println("After modification, str:  ", string(rune_str))

	// 单引号中的字符常量为 `rune` 类型，取类型后为 int32
	v_char := 'g'
	fmt.Printf("The type of v_char is %T\n", v_char)

}
{% endhighlight %}

将上面的代码存入源文件 string.go 并使用 `go run string.go` 可以看到下面的输入：

{% highlight text %}
emptyStr is:  
len(emptyStr) is:  0
string object is:  I like 高圆圆
len(str) = :  16
str[1]:  32
str:  I
      Love
      Golang
Before modification, str:  I like 高圆圆
len(str) = :  16
len([]rune(str)) =  10
len([]byte(str)) =  16
After modification, str:   I like 范冰冰
The type of v_char is int32
{% endhighlight %}

## 指针类型 `pointer`

Golang 中的指针和 C/C++ 相同的地方有：

- 指针 *T
- 指针的指针 **T
- 通过 operator `*` 访问指针指向的对象，`&` 取对象的地址

不同的地方有：

- 指针的默认值为 `nil`，而不是 `NULL`
- 指针不能进行加减运算，指针所指对象的成员使用 `.` 访问而非 `->`
- 使用 `unsafe.Pointer` 可以对指向不同类型对象的指针进行转换
- 指针不能进行 `+`、`-` 运算，以保证安全性。

使用实例如下：

{% highlight go %}
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	// 检查指针对象的零值为 nil
	var p *int
	fmt.Println("The zero value of a pointer is: ", p)

	// 指向指针的指针
	pp := &p
	fmt.Printf("The type of a pointer points another pointer is: %T\n", pp)

	// 指针对象赋值
	intVar := 100000000
	p = &intVar
	fmt.Println("After assignment, p is: ", p)
	fmt.Println("The value pointer p points is: ", *p)

	// 使用 unsafe.Pointer 方法可以将一个类型的指针转化为 Pointer
	// Pointer 可以被转化为任意类型的指针。
	// 注意由于 int 为 int32 的别名，占 4 个字节，所以我们将其转化为含有 4 个字节元素的 `byte` 数组指针
	var strP *[4]byte
	strP = (*[4]byte)(unsafe.Pointer(p))
	fmt.Println("After \"(*[4]byte)(unsafe.Pointer(p))\", *[4]byte pointer strP is: ", strP)
	fmt.Println("After \"(*[4]byte)(unsafe.Pointer(p))\", *[4]byte pointer strP points to: ", *strP)

	// 指针指向的对象内容使用 `.` 而不是 `->` 来进行访问
	type User struct {
		name string
	}
	userP := &User{
		"Xiaohui",
	}
	fmt.Println("Before change, The value userP points to is: ", *userP)
	userP.name = "Ross"
	fmt.Println("After change,  The value userP points to is: ", *userP)

}

{% endhighlight %}

将上面的代码存入源文件 pointer.go 并使用 `go run pointer.go` 可以看到下面的输入：

{% highlight text %}
The zero value of a pointer is:  <nil>
The type of a pointer points another pointer is: **int
After assignment, p is:  0xc20800a240
The value pointer p points is:  100000000
After "(*[4]byte)(unsafe.Pointer(p))", *[4]byte pointer strP is:  &[0 225 245 5]
After "(*[4]byte)(unsafe.Pointer(p))", *[4]byte pointer strP points to:  [0 225 245 5]
Before change, The value userP points to is:  {Xiaohui}
After change,  The value userP points to is:  {Ross}
{% endhighlight %}

关于 `unsafe.Pointer`，建议阅读[这篇文章][1]，里面做了很好的总结。

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

[1]: http://learngowith.me/gos-pointer-pointer-type/


