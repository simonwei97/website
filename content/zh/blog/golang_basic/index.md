---
title: "Go基础知识总结"
subtitle: ""
date: 2022-03-02T17:51:01+08:00
draft: false
# page title background image
bg_image: "images/backgrounds/page-title.jpg"
# about image
image: "images/blog/golang.jpg"
# meta description
description: "Go基础知识总结"
# categories
categories: ["go"]
# tags
tags: ["go"]
# type
type: "post"

aliases: ""
---

## 1.Channel 通道

一个通道相当于一个先进先出（`FIFO`）的队列。也就是说，通道中的各个元素值都是严格地按照发送的顺序排列的，先被发送通道的元素值一定会先被接收。元素值的发送和接收都需要用到操作符`<-`。我们也可以叫它接送操作符。一个左尖括号紧接着一个减号形象地代表了元素值的传输方向。

-   **元素值从外界进入通道时会被复制。更具体地说，进入通道的并不是在接收操作符右边的那个元素值，而是它的副本。**
-   对于通道中的同一个元素值来说，发送操作和接收操作之间也是**互斥**的

**长时间阻塞：**

-   缓冲通道：如果通道已满，那么对它的发送操作都会被阻塞，直到通道内有元素被接收（异步）
-   非缓冲通道：用同步的方式传递数据。只有收发双方对接上了，数据才会被传递。
-   对于值为 `nil` 的通道，不论它的具体类型是什么，对它的发送操作和接收操作都会**永久地处于阻塞状态**。它们所属的 `goroutine` 中的任何代码，都不再会被执行。
-   向已经关闭的 `channel` 发送数据
-   关闭一个已经关闭的 `channel`

{{<callout note "注意">}}
由于通道类型是引用类型，所以它的零值就是`nil`。换句话说，当我们只声明该类型的变量但没有用`make`函数对它进行初始化时，该变量的值就会是`nil`。我们一定不要忘记初始化通道！引发 panic
{{</callout>}}

应当让发送方关闭通道。

## 2. 逃逸分析

-   逃逸分析的好处是为了减少 GC 的压力，不逃逸的对象分配在栈上，当函数返回时就回收了资源，不需要 GC 标记清除。
-   逃逸分析完后可以确定哪些变量可以分配在栈上，栈的分配比堆快，性能好（逃逸的局部变量会分配在堆上，而没有发送逃逸的则有编辑器分配到栈上）
-   同步消除，如果定义的对象在方法上有同步锁，但在运行时，却只有一个线程在访问，此时逃逸分析的机器码，发去掉同步锁进行。

### 2.1. 分析命令

```go
// -m打印逃逸分析信息，-l禁止内联编译
go run -gcflags "-m -l"

/*
    testProj go run -gcflags "-m -l" internal/test1/main.go
    # command-line-arguments
    internal/test1/main.go:4:2: moved to heap: a
    internal/test1/main.go:5:11: main make([]*int, 1) does not escape
*/

// 汇编代码中搜runtime.newobject指令，该指令用于生成堆对象
go tool compile -S main.go | grep runtime.newobject

/*
    testProj go tool compile -S internal/test1/main.go | grep newobject
        0x0028 00040 (internal/test1/main.go:4) CALL    runtime.newobject(SB)
*/
```

-   申请到`栈内存`好处：函数返回直接释放，不会引起垃圾回收，对性能没有影响。
-   申请到`堆上面的内存`才会引起垃圾回收，如果这个过程（特指垃圾回收不断被触发）过于高频就会导致 gc 压力过大，程序性能出问题。

### 2.2. 逃逸场景（什么情况才分配到堆中）

1. 指针逃逸
2. 栈空间不足逃逸（空间开辟过大）
3. 动态类型逃逸（不确定长度大小）
4. 闭包引用对象逃逸

{{<callout note "以下情况一定发生指针逃逸">}}

-   在某个函数中 new 或字面量创建出的变量，将其指针作为函数返回值，则该变量一定发生逃逸（构造函数返回的指针变量一定逃逸）；
-   被已经逃逸的变量引用的指针，一定发生逃逸。
-   被指针类型的 slice、map 和 chan 引用的指针一定发生逃逸

{{</callout>}}

备注：stack overflow 上有人提问为什么使用指针的 chan 比使用值的 chan 慢 30%，答案就在这里：使用指针的 chan 发生逃逸，gc 拖慢了速度。[why-passing-pointers-to-channel-is-slower](https://stackoverflow.com/questions/41178729/why-passing-pointers-to-channel-is-slower)

**一些必然不会逃逸的情况**

-   指针被未发生逃逸的变量引用；
-   仅仅在函数内对变量做取址操作，而未将指针传出；

**有一些情况可能发生逃逸，也可能不会发生逃逸**

-   将指针作为入参传给别的函数；这里还是要看指针在被传入的函数中的处理过程，如果发生了上边的三种情况，则会逃逸；否则不会逃逸；

## 3.defer 语句

-   **多个 defer 语句执行顺序**

在 defer 语句每次执行的时候，Go 语言会把它携带的 defer 函数及其参数值另行存储到一个队列中。这个队列与该 defer 语句所属的函数是对应的，并且，它是先进后出（FILO）的，相当于一个栈。即**defer 函数调用与其所属的 defer 语句的执行顺序完全相反**。

-   **defer 与 return 谁先谁后**

return 之后的语句先执行，defer 后的的语句后执行

-   **函数的返回值初始化**

```go
package main

import "fmt"

func DeferFunc1(i int) (t int) {
    fmt.Println("t = ", t)
    return 2
}

func main() {
    DeferFunc1()
}
```

```bash
# 运行结果
t = 0
```

只要声明函数的返回值变量名称，就会在函数初始化时候为之赋值为 0，而且在函数体作用域可见。

## 4.make 和 new

-   **new**

```go
func new(Type) *Type
```

`new` 只接收一个参数，这个参数是一个类型，分配好内存后，返回一个指向该类型内存地址的指针。返回的永远是类型的指针，指向分配类型的内存地址。

-   **make**

```go
func make(t Type, size ...IntegerType) Type
```

`make` 用于分配内存，只用于 `chan`, `map`, `slice` 的内存创建，而且它返回的类型就是这三个类型的本身，而不是它们的指针类型，因为这三种类型是引用类型，没必要返回它们的指针了。

## 5.参考链接

-   [golang 逃逸分析详解](https://zhuanlan.zhihu.com/p/91559562)
-   [Golang 内存分配逃逸分析](https://driverzhang.github.io/post/golang%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90/)
-   [Go 的 sync](https://driverzhang.github.io/post/go%E7%9A%84sync.pool%E4%B8%B4%E6%97%B6%E5%AF%B9%E8%B1%A1%E6%B1%A0/)
