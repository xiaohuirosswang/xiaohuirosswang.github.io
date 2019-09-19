---
layout: post
title: "Golang 语言基础之三： array, slice"
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

## 数组

Golang 语言中，一个数组是一组已被编号的属于同一类型的元素。数组中元素的个数就是数组的长度。数组的类型由数组中元素的类型和数组的长度共同定义，数组在未指定长度时默认长度为零。数组元素的索引从 `0` 到 `len(ArrayObject) - 1` 或者说 `cap(ArrayObject) - 1`。

还是从例子来看：

{% highlight go %}
package main

import "fmt"

// 值传递方式传递数组对象
func arrayTestByValue(arr [1]int) {
	fmt.Println("Passed by value, Address of arr[0] is: ", &arr[0])
}

// 引用传递方式传递数组对象
func arrayTestByref(arr *[1]int) {
	fmt.Println("Passed by ref, Address of arr[0] is: ", &arr[0])
}

func main() {

	// 定义空数组，方括号中的 ... 不能省略，否则就是 slice 对象了。
	v_IntArray := [...]int{}
	fmt.Printf("%T\n", v_IntArray)
	fmt.Println("len(v_IntArray): ", len(v_IntArray))
	fmt.Println("cap(v_IntArray): ", cap(v_IntArray))

	// 声明定长数组，如果没有给定初始值，默认初始化元素为零值，对于 int 类型，就是 0
	v_IntArrayOf5 := [5]int{}
	fmt.Println("len(v_IntArrayOf5: ", len(v_IntArrayOf5))
	fmt.Println("v_IntArrayOf5: ", v_IntArrayOf5)
	v_IntArrayOf5[2] = 3
	fmt.Println("v_IntArrayOf5: ", v_IntArrayOf5)

	// 声明数组的同时进行初始化
	v_IntArrayOf5 = [5]int{1, 2, 3, 4, 5}
	fmt.Println("len(v_IntArrayOf5: ", len(v_IntArrayOf5))
	fmt.Println("v_IntArrayOf5: ", v_IntArrayOf5)

	// 可以给数组中的部分元素给定初始值，下面的方式中数组前两个元素会被初始化为指定值
	v_IntArrayOf5 = [5]int{1, 2}
	fmt.Println("len(v_IntArrayOf5: ", len(v_IntArrayOf5))
	fmt.Println("v_IntArrayOf5: ", v_IntArrayOf5)

	// 可以使用索引进行个别元素的初始化
	v_IntArrayOf5 = [5]int{1: 2, 3: 4}
	fmt.Println("len(v_IntArrayOf5: ", len(v_IntArrayOf5))
	fmt.Println("v_IntArrayOf5: ", v_IntArrayOf5)

	// 可以进行分段初始化
	v_IntArrayOf5 = [5]int{0: 1, 2, 3: 4, 5}
	fmt.Println("len(v_IntArrayOf5: ", len(v_IntArrayOf5))
	fmt.Println("v_IntArrayOf5: ", v_IntArrayOf5)

	// 数组进行参数传递的时候是值拷贝方式
	// arrayTestByValue 函数中的数组在内存中的地址和原始地址不相同了。
	// arrayTestByref 函数中数组在内存中的地址和原始数组相同
	v_IntArrayOf1 := [1]int{}
	fmt.Println("Original Address of v_IntArrayOf1[0] is: ", &v_IntArrayOf1[0])
	arrayTestByValue(v_IntArrayOf1)
	arrayTestByref(&v_IntArrayOf1)

	// Golang 支持多维数组，下面是 2 维数组的例子
	v_IntArrayOf23 := [2][3]int{{2}}
	fmt.Println("v_IntArrayOf23: ", v_IntArrayOf23)

}

{% endhighlight %}

将上面的代码存入源文件 array.go 并使用 `go run array.go` 可以看到下面的输入：

{% highlight text %}
len(v_IntArray):  0
cap(v_IntArray):  0
len(v_IntArrayOf5:  5
v_IntArrayOf5:  [0 0 0 0 0]
v_IntArrayOf5:  [0 0 3 0 0]
len(v_IntArrayOf5:  5
v_IntArrayOf5:  [1 2 3 4 5]
len(v_IntArrayOf5:  5
v_IntArrayOf5:  [1 2 0 0 0]
len(v_IntArrayOf5:  5
v_IntArrayOf5:  [0 2 0 4 0]
len(v_IntArrayOf5:  5
v_IntArrayOf5:  [1 2 0 4 5]
Original Address of v_IntArrayOf1[0] is:  0xc20800a3d8
Passed by value, Address of arr[0] is:  0xc20800a400
Passed by ref, Address of arr[0] is:  0xc20800a3d8
v_IntArrayOf23:  [[2 0 0] [0 0 0]]
{% endhighlight %}

关于数组需要注意的是：

- 由于数组长度是数组类型的组成部分，所以数组长度必须在声明数组的时候就是确定的常量。
- 数组赋值和数组进行参数传递时都会进行值拷贝，所以在参数传递时为了提高效率可以考虑使用 `slice` 或者数组指针。
- 在 [Golang 语言基础之二： for, ifelse, switch](/golang-fundamentals-2-for-ifelse-switch.html) 的 `for` 语句例子中，我们看到了如何定义数组指针 `*[]T` 和指针的数组 `[*]T`。
- 由于数据在声明时长度已经确定，Golang也会对数组中的元素进行初始化（默认初始值为零值），所以数组对象可以直接进行 `==` 和 `!=` 的逻辑比较。

## Slice （切片）

`slice` 是 Golang 中的一个重要数据类型，由前面对数组的讨论可知，数组在一些情况下（比如进行参数传递）使用不方便，数组的类型由数组元素的类型和数组长度确定，这点对使用其作为参数的函数来说限制太大。所以 Golang 中提供了 `slice` 数据结构，其是对相关数组的一个连续片段的引用，故而 `slice` 是一个引用类型。它的长度是可变的，从零到相关数组的长度。`len` 内置函数返回 `slice` 对象的当前长度，`cap` 内置函数返回 `slice` 对象的最大长度。

`slice` 在源码文件 `runtime.h` 中的定义为：

{% highlight c %}
struct Slice
{ 
	             // must not move anything
	byte* array; // actual data
	uintgo len;  // number of elements
	uintgo cap;  // allocated number of elements
};
{% endhighlight %}

看个例子：

{% highlight go %}
package main

import "fmt"

func sliceTest(sli []int) {
	fmt.Printf("Address of sli is: %d\n", &sli[0])
}

func main() {

	// 声明数组
	v_IntArray := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	fmt.Println("v_IntArray is: ", v_IntArray)

	// 对数组进行切片，方括号里面数字的意义为 `[low, high, capacity]`
	// 对应数组索引 `[starterIndex, endIndex + 1, lengthOfArray - starterIndex]`
	v_IntSlice := v_IntArray[1:4:9]
	fmt.Println("v_IntArray[1:4:9], v_IntSlice is: ", v_IntSlice)
	fmt.Println("v_IntArray[1:4:9], len(v_IntSlice) is: ", len(v_IntSlice))
	fmt.Println("v_IntArray[1:4:9], cap(v_IntSlice) is: ", cap(v_IntSlice))

	// 省略 `high` 和 `capacity`，省略 `high` 会从数组索引为 `low` 的元素一直到数组最后一个元素，切片的容量为 `lengthOfArray - starterIndex`
	v_IntSlice = v_IntArray[1:]
	v_IntSlice[0] = 100
	fmt.Println("v_IntArray is: ", v_IntArray)
	fmt.Println("v_IntArray[1:], v_IntSlice is: ", v_IntSlice)
	fmt.Println("v_IntArray[1:], len(v_IntSlice) is: ", len(v_IntSlice))
	fmt.Println("v_IntArray[1:], cap(v_IntSlice) is: ", cap(v_IntSlice))

	// 省略 `low` 和 `capacity`，省略 `low` 会从数组索引为 `0` 的元素一直到索引为 `high` 的元素
	// 切片的容量和数组长度相等
	v_IntSlice = v_IntArray[:1]
	fmt.Println("v_IntArray[:1], v_IntSlice is: ", v_IntSlice)
	fmt.Println("v_IntArray[:1], len(v_IntSlice) is: ", len(v_IntSlice))
	fmt.Println("v_IntArray[:1], cap(v_IntSlice) is: ", cap(v_IntSlice))

	// 全部省略，从数组索引为 `0` 的元素一直到数组最后一个元素，切片的容量和数组长度相等
	v_IntSlice = v_IntArray[:]
	fmt.Println("v_IntArray[:], v_IntSlice is: ", v_IntSlice)
	fmt.Println("v_IntArray[:], len(v_IntSlice) is: ", len(v_IntSlice))
	fmt.Println("v_IntArray[:], cap(v_IntSlice) is: ", cap(v_IntSlice))

	// 省略 `low`，省略 `low` 会从数组索引为 `0` 的元素一直到索引为 `high` 的元素
	// 切片对象的 `capacity` 为给定的数值，注意这个数值不能大于数组的长度 
	v_IntSlice = v_IntArray[:4:4]
	fmt.Println("v_IntArray[:4:4], v_IntSlice is: ", v_IntSlice)
	fmt.Println("v_IntArray[:4:4], len(v_IntSlice) is: ", len(v_IntSlice))
	fmt.Println("v_IntArray[:4:4], cap(v_IntSlice) is: ", cap(v_IntSlice))

	// 如果所有初始值已经确定，可以这样直接声明 `slice` 对象
	v_IntSlice = []int{1, 2, 5: 6}
	v_IntSlice[0] = 100
	fmt.Println("v_IntArray is: ", v_IntArray)
	fmt.Println("[]int{1, 2, 5: 6}, v_IntSlice is: ", v_IntSlice)

	// 如果需要之后进行 `slice` 对象赋值，可以使用 `make` 内置函数声明 `slice` 对象
	v_IntSlice = make([]int, 5, 10)
	fmt.Println("make([]int, 5, 10), v_IntSlice is: ", v_IntSlice)
	fmt.Println("make([]int, 5, 10), len(v_IntSlice) is: ", len(v_IntSlice))
	fmt.Println("make([]int, 5, 10), cap(v_IntSlice) is: ", cap(v_IntSlice))

	// 使用 `make` 内置函数声明 `slice` 对象的时候可以省略 `capacity`
	// 其默认值和给定的 `slice` 的 `length` 相等。
	v_IntSlice = make([]int, 5)
	fmt.Println("make([]int, 5), v_IntSlice is: ", v_IntSlice)
	fmt.Println("make([]int, len(v_IntSlice) is: ", len(v_IntSlice))
	fmt.Println("make([]int, cap(v_IntSlice) is: ", cap(v_IntSlice))

	// 使用 `new` 内置函数也可以声明 `slice` 对象
	v_IntSlice = new([10]int)[:5]
	fmt.Println("new([10]int)[:5], v_IntSlice is: ", v_IntSlice)
	fmt.Println("new([10]int)[:5], len(v_IntSlice) is: ", len(v_IntSlice))
	fmt.Println("new([10]int)[:5], cap(v_IntSlice) is: ", cap(v_IntSlice))

	// 可以对切片进行再切片
	v_IntSlice = make([]int, 5)
	v_AnotherIntSlice := v_IntSlice[3:]
	v_AnotherIntSlice[0] = 100
	fmt.Println("v_AnotherIntSlice = v_IntSlice[3:], len(v_AnotherIntSlice) is: ", len(v_AnotherIntSlice))
	fmt.Println("v_AnotherIntSlice = v_IntSlice[3:], cap(v_AnotherIntSlice) is: ", cap(v_AnotherIntSlice))
	fmt.Println("v_AnotherIntSlice = v_IntSlice[3:], v_IntSlice is: ", v_IntSlice)
	fmt.Println("v_AnotherIntSlice = v_IntSlice[3:], v_AnotherIntSlice is: ", v_AnotherIntSlice)

	// 使用 `append` 内置函数可以对 `slice` 对象在其容量范围内进行添加，新添加的元素索引为 `len(切片对象)`，操作返回新的切片对象
	v_IntSlice = make([]int, 1, 2)
	v_AnotherIntSlice = append(v_IntSlice, 100)
	fmt.Println("v_AnotherIntSlice = append(v_IntSlice, 100), v_IntSlice is: ", v_IntSlice)
	fmt.Println("v_AnotherIntSlice = append(v_IntSlice, 100), v_AnotherIntSlice is: ", v_AnotherIntSlice)
	fmt.Printf("Address of v_IntSlice is: %d\n", &v_IntSlice[0])
	fmt.Printf("Address of v_AnotherIntSlice is: %d\n", &v_AnotherIntSlice[0])

	// 如果使用 `append` 追加元素的时候超出了切片对象的容量，Golang 会重新创建一个匿名数组来保存新的切片对象中的数据
	v_AnotherIntSlice = append(v_IntSlice, 100, 200)
	fmt.Println("v_AnotherIntSlice = append(v_IntSlice, 100, 200), v_IntSlice is: ", v_IntSlice)
	fmt.Println("v_AnotherIntSlice = append(v_IntSlice, 100, 200), v_AnotherIntSlice is: ", v_AnotherIntSlice)
	fmt.Printf("Address of v_IntSlice is: %d\n", &v_IntSlice[0])
	fmt.Printf("Address of v_AnotherIntSlice is: %d\n", &v_AnotherIntSlice[0])

	// 使用切片对象作为函数参数传递时是值传递，但是由于 `slice` 是引用类型，所以不会拷贝相关数组的值
	v_IntSlice = make([]int, 5, 10)
	fmt.Printf("Address of v_IntSlice is: %d\n", &v_IntSlice[0])
	sliceTest(v_IntSlice)

}

{% endhighlight %}

将上面的代码存入源文件 slice.go 并使用 `go run slice.go` 可以看到下面的输入：

{% highlight text %}
v_IntArray is:  [0 1 2 3 4 5 6 7 8 9]
v_IntArray[1:4:9], v_IntSlice is:  [1 2 3]
v_IntArray[1:4:9], len(v_IntSlice) is:  3
v_IntArray[1:4:9], cap(v_IntSlice) is:  8
v_IntArray is:  [0 100 2 3 4 5 6 7 8 9]
v_IntArray[1:], v_IntSlice is:  [100 2 3 4 5 6 7 8 9]
v_IntArray[1:], len(v_IntSlice) is:  9
v_IntArray[1:], cap(v_IntSlice) is:  9
v_IntArray[:1], v_IntSlice is:  [0]
v_IntArray[:1], len(v_IntSlice) is:  1
v_IntArray[:1], cap(v_IntSlice) is:  10
v_IntArray[:], v_IntSlice is:  [0 100 2 3 4 5 6 7 8 9]
v_IntArray[:], len(v_IntSlice) is:  10
v_IntArray[:], cap(v_IntSlice) is:  10
v_IntArray[:4:4], v_IntSlice is:  [0 100 2 3]
v_IntArray[:4:4], len(v_IntSlice) is:  4
v_IntArray[:4:4], cap(v_IntSlice) is:  4
v_IntArray is:  [0 100 2 3 4 5 6 7 8 9]
[]int{1, 2, 5: 6}, v_IntSlice is:  [100 2 0 0 0 6]
make([]int, 5, 10), v_IntSlice is:  [0 0 0 0 0]
make([]int, 5, 10), len(v_IntSlice) is:  5
make([]int, 5, 10), cap(v_IntSlice) is:  10
make([]int, 5), v_IntSlice is:  [0 0 0 0 0]
make([]int, len(v_IntSlice) is:  5
make([]int, cap(v_IntSlice) is:  5
new([10]int)[:5], v_IntSlice is:  [0 0 0 0 0]
new([10]int)[:5], len(v_IntSlice) is:  5
new([10]int)[:5], cap(v_IntSlice) is:  10
v_AnotherIntSlice = v_IntSlice[3:], len(v_AnotherIntSlice) is:  2
v_AnotherIntSlice = v_IntSlice[3:], cap(v_AnotherIntSlice) is:  2
v_AnotherIntSlice = v_IntSlice[3:], v_IntSlice is:  [0 0 0 100 0]
v_AnotherIntSlice = v_IntSlice[3:], v_AnotherIntSlice is:  [100 0]
v_AnotherIntSlice = append(v_IntSlice, 100), v_IntSlice is:  [0]
v_AnotherIntSlice = append(v_IntSlice, 100), v_AnotherIntSlice is:  [0 100]
Address of v_IntSlice is: 833357914768
Address of v_AnotherIntSlice is: 833357914768
v_AnotherIntSlice = append(v_IntSlice, 100, 200), v_IntSlice is:  [0]
v_AnotherIntSlice = append(v_IntSlice, 100, 200), v_AnotherIntSlice is:  [0 100 200]
Address of v_IntSlice is: 833357914768
Address of v_AnotherIntSlice is: 833357996192
Address of v_IntSlice is: 833358192800
Address of sli is: 833358192800
{% endhighlight %}

`slice` 的具体用法和注意事项在上面实例中注释部分，稍多，但是作为 Golang 的重要数据类型，需要完全掌握。

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



