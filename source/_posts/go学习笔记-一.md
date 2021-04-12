---
title: go学习笔记<一>
date: 2021-04-05 11:26:30
thumbnail: https://img.szpinc.org/2021/04/05/11/kWDvD9.jpg
tags:
    - go
categories:
    - go学习笔记
---

让我们从 Go（或 Golang）的一个小介绍开始。 Go 由 Google 工程师 Robert Griesemer，Rob Pike 和 Ken Thompson 设计。 它是一种静态类型的编译语言。 第一个版本于 2012 年 3 月作为开源发布。

> “Go 是一种开源编程语言，可以轻松构建的简单，可靠，高效的软件”。

关于 go: 

在许多语言中，有许多方法可以解决某些给定的问题。因此 程序员可以花很多时间思考解决问题的最佳方法。

然而，Go 却是只有一种正确的方法来解决问题的语言。

这节省了开发人员的时间，并使大型代码库易于维护。 Go 中没有地图和过滤器等 “富有表现力” 的功能。

> “如果你有增加表现力的功能，通常会增加费用” — Rob Pike


## 入门

Go 是由包组成的。 main 包告诉 Go 编译器该程序可以被编译成可执行文件，而不是一个共享的库。它是应用程序的入口。main 包被定义为如下格式:

``` go
package main
```

接下来，让我们通过在 Go 工作区中创建一个文件 main.go 来编写一个简单的 hello world 示例。

### go 的工作区

Go 中的工作空间由环境变量「GOPATH」定义。你写的任何代码都将写在工作区内。Go 将搜索 `GOPATH` 目录中的任何包，或者在安装 Go 时默认设置的 `GOROOT` 目录。 `GOROOT` 是安装 go 的路径。

将 `GOPATH` 设置为你想要的目录。 现在，让我们将它添加到文件夹`〜/workspace`中。

``` bash
# 写入 env
export GOPATH=~/workspace

# cd 到工作区目录\
cd ~/workspace
```

使用我们刚刚创建的工作空间文件夹中的以下代码创建文件 `main.go`。

### Hello World!

``` go
package main

import "fmt"

func main(){
  fmt.Println("Hello World!")
}
```

在上面的 demo 中，`fmt` 是 Go 中的内置包，它实现了格式化I/O的功能。

** 编译和运行go程序: **
``` bash
# 编译go文件，会生成一个可执行文件main
go build main.go
# 执行main文件
./main
```

也可以合并到一起:
``` bash
# 相当于执行上面的两步操作
go run main.go
```

## 变量

变量在`Go`语言中是一个很明确的定义。`Go`是一种静态类型的语言。这意味着在声明变量时我们就需要明确变量的类型。一般一个变量的定义如下:

``` go
var a int
```

上面的实例中，我们定义了一个`int`类型的变量`a`，默认会被赋值成`0`。使用以下语法可以初始化改变变量的值:

``` go
var a = 1
```

这里我们没有制定变量`a`的类型，在我们给它初始化为`1`时，它就自动被定义成了`int`类型的变量。我们也可以使用一种更简短的语法来定义它:

``` go
message := "hello world"
```

我们也可以在同一行声明多个同类型变量:

``` go
var b, c int = 2, 3
```

## 数据类型

跟任何其他编程语言一样，`Go`语言支持各种不同的数据结构。 让我们探讨一下：

### Number, String, and Boolean

一些受支持的`number`存储类型是：int, int8, int16, int32, int64,
uint, uint8, uint16, uint32, uint64, uintptr…

`string`类型存储一些列的字节。 它用关键字`string`表示和声明。

`boolean`使用关键字`bool`存储布尔值。

`Go`还支持复数类型数据类型，可以使用`complex64`和`complex128`声明。

``` go
var a bool = true
var b int = 1
var c string = 'hello world'
var d float32 = 1.222
var x complex128 = cmplx.Sqrt(-5 + 12i)
```

### 数组，切片，以及 Maps

数组是相同数据类型的元素序列。 数组在声明中定义要指定长度，因此不能进行扩展。 数组声明为：

``` go
var a [5]int
```

数组也可以是多维的。 我们可以使用以下格式创建它们：

``` go
var multiD [2][3]int
```

当数组的值在运行时不能进行更改。 数组也不提供获取子数组的能力。 为此，Go 有一个名为切片的数据类型。
切片存储一系列元素，可以随时扩展。 切片声明类似于数组声明 — 没有定义容量：

``` go
var b []int
```

这将创建一个零容量和零长度的切片。 切片也可以定义容量和长度。 我们可以使用以下语法：

``` go
numbers := make([]int,5,10)
```


