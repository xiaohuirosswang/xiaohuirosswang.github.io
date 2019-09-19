---
layout: post
title: "Golang 语言基础之十： goroutine, channel"
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

Quora 上有一个问题：[What is difference between Goroutines vs OS threads?][1]，相信这是也很多学习 Golang 的程序员比如我自己关心的问题。这个问题下的这个答案说得比较清楚，其作者自称来自 Google 的 Golang Team：

> The Go runtime multiplexes a potentially large number of goroutines onto a smaller number of OS threads, and goroutines blocked on I/O are handled efficiently using epoll or similar facilities.  Goroutines have tiny stacks that grow as needed, so it is practical to have hundreds of thousands of goroutines in your program. This allows the programmer to use concurrency to structure their program without being overly concerned with thread overhead.[Why goroutines instead of threads?][3]

结合 [官方文档中对 goroutine 的定义][2]：

> They're called goroutines because the existing terms—threads, coroutines, processes, and so on—convey inaccurate connotations. A goroutine has a simple model: it is a function executing concurrently with other goroutines in the same address space. It is lightweight, costing little more than the allocation of stack space. And the stacks start small, so they are cheap, and grow by allocating (and freeing) heap storage as required.

[学习 Go 语言  中文版](http://mikespook.com/learning-go/) 对这段定义的翻译为：

> 叫做 goroutine 是因为已有的短语——线程、协程、进程等等——传递了不准确的含义。goroutine 有简单的模型：它是与其他goroutine 并行执行的，有着相同地址空间的函数。它是轻量的，仅比分配栈空间多一点点，而初始时栈是很小的，所以它们也是廉价的，并且随着需要在堆空间上分配（和释放）。

简单地总结一下，`goroutine` 是 Golang 中从语言层面支持并发的重要特性，运行在操作系统线程中的轻量级 `coroutine`。

我们可以在函数或者方法前面加 `go` 关键字来创建一个并发执行单元。Golang 的调度器不保证多个 goroutine 的执行顺序，Golang 应用程序进程退出的时候也不会检查是否还有未结束的 `goroutine`，所以，我们需要在代码中自己管理 `goruntine` 的生命周期。

`goroutine` 中运行函数的值及其参数和正常的函数调用基本是一样的，不同的地方在于当前操作不需要等待启动的 `goroutine` 的返回值。`goroutine` 中运行的函数执行完毕后，这个 `goroutine` 也就结束了。如果这里的函数有返回值的话，会被丢弃。

{% highlight go %}
package main

import (
	"fmt"
	"runtime"
	"sync"
)

// 定义外部函数
func FuncA(wg *(sync.WaitGroup)) {
	defer wg.Done()
	fmt.Println("This is FuncA.")
}

// 定义一个结构类型并为其定义一个方法
type DummyStruct struct{}

func (ds *DummyStruct) FuncB(wg *(sync.WaitGroup)) {
	defer wg.Done()
	fmt.Println("This is DummyStruct.FuncB().")
}

// 当以 goroutine 方式运行时主动中断自身
func FuncWithTermination(wg *(sync.WaitGroup)) {
	defer func() {
		fmt.Println("defer in FuncWithTermination")
		wg.Done()
	}()
	runtime.Goexit()
	fmt.Println("This is FuncWithTermination.")
}

// 当以 goroutine 方式运行时暂停自身之后系统线程空闲后再恢复运行
func FuncWithGosched(wg *(sync.WaitGroup)) {
	defer wg.Done()
	for index := 0; index < 5; index++ {
		fmt.Println("Count ", index)
		if index == 2 {
			runtime.Gosched()
		}
	}
}

func main() {
	// goroutine 的个数
	grCount := 5

	// 设置可以并行运行的核心数
	runtime.GOMAXPROCS(1)

	// 等待 goroutine 执行结果
	wg := new(sync.WaitGroup)
	wg.Add(grCount)
	fmt.Println("Starting...")
	go FuncWithGosched(wg)
	go FuncA(wg)
	ds := DummyStruct{}
	go ds.FuncB(wg)
	go func(wg *(sync.WaitGroup)) {
		defer wg.Done()
		fmt.Println("This is inner anonymous func.")
	}(wg)
	go FuncWithTermination(wg)
	wg.Wait()
}
{% endhighlight %}

将上面的代码存入源文件 goroutine.go 并使用 `go run goroutine.go` 可以看到下面的输入：

{% highlight text %}
Starting...
Count  0
Count  1
Count  2
This is FuncA.
This is DummyStruct.FuncB().
This is inner anonymous func.
defer in FuncWithTermination
Count  3
Count  4
{% endhighlight %}

需要注意的是：

- `go` 关键字后可以创建执行函数和对象方法的 `goroutine`。
- `go` 支持匿名函数
- 如果程序进程在所有 `goroutine` 执行完毕前结束，则这些 `goroutine` 也会被全部销毁。
- 在 `goroutine` 执行过程中调用 `runtime.Goexit()` 可以结束该 `goroutine` ，Golang 会保证已经定义的 defer 函数按照 FILO 规则运行。
- 在 `goroutine` 执行过程中调用 `runtime.Gosched()` 可以暂停自身运行，当系统线程空闲时恢复运行。
- 

## channel

[官方文档中关于 channel 的定义][4] 如下：

> A channel provides a mechanism for concurrently executing functions to communicate by sending and receiving values of a specified element type. The value of an uninitialized channel is nil.

`channel` 类型对象通过传递 `send` 和 `receive` 特定类型的值而提供了并行执行的函数之间相互通信的机制。没有指定初始值的 `channel` 类型对象默认值为 `nil`。比如：

> The optional <- operator specifies the channel direction, send or receive. If no direction is given, the channel is bidirectional. A channel may be constrained only to send or only to receive by conversion or assignment.

默认情况下一个 `channel` 对象是双向的，可以发送和接收数据；通过可选的 `<-` 操作符可以将 `channel` 类型对象相应限制为 `只发送` 和 `只接收`。比如：

{% highlight go %}
chan T          // can be used to send and receive values of type T
chan<- float64  // can only be used to send float64s
<-chan int      // can only be used to receive ints
{% endhighlight %}

`<-` 尽量地和左边的 `channel` 类型对象结合：

{% highlight go %}
chan<- chan int    // same as chan<- (chan int)
chan<- <-chan int  // same as chan<- (<-chan int)
<-chan <-chan int  // same as <-chan (<-chan int)
chan (<-chan int)
{% endhighlight %}

> A new, initialized channel value can be made using the built-in function make, which takes the channel type and an optional capacity as arguments:

我们必须使用内置函数 `make` 来创建一个 `channel` 类型对象。`make` 函数的参数依次为：`channel` 类型名、该类型接受的值的类型以及可选的 `channel` buffer值。通过内置函数 `len` 以及 `cap` 可以获取到一个 `channel` 类型对象的 buffer 值。当buffer 值不为零时，该 `channel` 类型对象在达到 buffer 值之前只能发送，接收被 block，达到 buffer 值之后只能接收，发送被 block。

	make(chan int, 100)

让我们看看如何使用 `channel`：

{% highlight go %}
package main

import (
	"fmt"
	"time"
)

func main() {

	// 简单使用 channel 的例子
	varChan1 := make(chan string)
	go func() {
		varChan1 <- "I am a string."
	}()
	fmt.Println("Got msg: ", <-varChan1)

	// 创建 channel 类型对象时设置了 buffer 值
	varChan2 := make(chan string, 3)
	go func() {
		varChan2 <- "input 1"
		varChan2 <- "input 2"
		fmt.Println("I will be before all inputs")
		varChan2 <- "input 3"
	}()
	fmt.Println("Got msg: ", <-varChan2)
	fmt.Println("Got msg: ", <-varChan2)
	fmt.Println("Got msg: ", <-varChan2)

	// channel 中的同步机制
	varChan3 := make(chan bool, 1)
	go func(varChan chan bool) {
		fmt.Println("begin to execute")
		// do something
		time.Sleep(time.Second * 2)
		fmt.Println("end")
		varChan <- true
	}(varChan3)
	fmt.Println("Got msg: ", <-varChan3)

	// 单向 channel 类型对象的使用
	inChan := make(chan string, 1)
	outChan := make(chan string, 1)
	inChan <- "Input str"
	anonymousFunc := func(varInChan chan<- string, varOutChan <-chan string) {
		varInChan <- <-varOutChan
	}
	anonymousFunc(outChan, inChan)
	fmt.Println("Got msg: ", <-outChan)

	// https://gobyexample.com/select
	c1 := make(chan string)
	c2 := make(chan string)
	go func() {
		time.Sleep(time.Second * 1)
		c1 <- "one"
	}()
	go func() {
		time.Sleep(time.Second * 2)
		c2 <- "two"
	}()
	for i := 0; i < 2; i++ {
		select {
		case msg1 := <-c1:
			fmt.Println("received", msg1)
		case msg2 := <-c2:
			fmt.Println("received", msg2)
		}
	}

	// https://gobyexample.com/timeouts
	c3 := make(chan string, 1)
	go func() {
		time.Sleep(time.Second * 2)
		c3 <- "result 1"
	}()
	select {
	case res := <-c3:
		fmt.Println(res)
	case <-time.After(time.Second * 1):
		fmt.Println("timeout 1")
	}
	c4 := make(chan string, 1)
	go func() {
		time.Sleep(time.Second * 2)
		c4 <- "result 2"
	}()
	select {
	case res := <-c4:
		fmt.Println(res)
	case <-time.After(time.Second * 3):
		fmt.Println("timeout 2")
	}

	// https://gobyexample.com/non-blocking-channel-operations
	messages := make(chan string)
	signals := make(chan bool)
	select {
	case msg := <-messages:
		fmt.Println("received message", msg)
	default:
		fmt.Println("no message received")
	}
	msg := "hi"
	select {
	case messages <- msg:
		fmt.Println("sent message", msg)
	default:
		fmt.Println("no message sent")
	}
	select {
	case msg := <-messages:
		fmt.Println("received message", msg)
	case sig := <-signals:
		fmt.Println("received signal", sig)
	default:
		fmt.Println("no activity")
	}

	// https://gobyexample.com/closing-channels
	jobs := make(chan int, 5)
	done := make(chan bool)

	go func() {
		for {
			j, more := <-jobs
			if more {
				fmt.Println("received job", j)
			} else {
				fmt.Println("received all jobs")
				done <- true
				return
			}
		}
	}()
	for j := 1; j <= 3; j++ {
		jobs <- j
		fmt.Println("sent job", j)
	}
	close(jobs)
	fmt.Println("sent all jobs")
	<-done

	// https://gobyexample.com/range-over-channels
	queue := make(chan string, 2)
	queue <- "one"
	queue <- "two"
	close(queue)
	for elem := range queue {
		fmt.Println(elem)
	}
}
{% endhighlight %}

将上面的代码存入源文件 channel.go 并使用 `go run channel.go` 可以看到下面的输入：

{% highlight text %}
Got msg:  I am a string.
I will be before all inputs
Got msg:  input 1
Got msg:  input 2
Got msg:  input 3
begin to execute
end
Got msg:  true
Got msg:  Input str
received one
received two
timeout 1
result 2
no message received
no message sent
no activity
sent job 1
sent job 2
sent job 3
sent all jobs
received job 1
received job 2
received job 3
received all jobs
one
two
{% endhighlight %}

这里引用了一些 [Go by Example](https://gobyexample.com/hello-world) 上 `channel` 类型相关的例子，作者注释的很充分，地址在注释中。

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

[1]: http://www.quora.com/What-is-difference-between-Goroutines-vs-OS-threads
[2]: https://golang.org/doc/effective_go.html#concurrency
[3]: http://golang.org/doc/faq#goroutines
[4]: http://golang.org/ref/spec#Channel_types


