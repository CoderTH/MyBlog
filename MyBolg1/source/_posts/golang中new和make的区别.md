---
title: golang中new和make的区别
tags: golang
categories: golang
index_img: /img/golang中new和make.jpg
banner_img: /img/golang中new和make.jpg
abbrlink: 55439
date: 2020-09-19 16:52:32
---

# golang中new和make的区别

### 前言
> 最近在学习golang中遇到一个小问题，网课中在给类型分配内存空间的时候大量的使用到了**new**和**make**关键字，但是对于他们两个的区别以及使用场景并没有过多的描述，对此我写下这篇博客以便于学习以及以后的复盘

### 简介
我们知道**new**与**make**关键字是用来分配内存空间的，在我们定义变量的时候，可能会觉得有点迷惑，不知道应该使用哪个函数来声明变量，其实他们的规则很简单，new 只分配内存，而 make 只能用于 slice、map 和 channel 的初始化，下面我们就来具体介绍一下

### new
我们首先看一下源码中对new关键字的描述：
```go
// The new built-in function allocates memory. The first argument is a type,
// not a value, and the value returned is a pointer to a newly
// allocated zero value of that type.
func new(Type) *Type
```
>新的内置函数分配内存。第一个参数是类型，不是值，返回的值是指向新值的指针分配了该类型的零值。
>根据文档我们可以知道**new**的主要作用就是给类型变量分配内存空间，他的第一个参数是类型，而不是具体的值，并且他的返回值是该类型的指针，指针指向的新值为该类型的零值
>下面是用**new**分配内存空间的小案例
```go
var number *int
number = new(int) //分配空间
*number = 98
fmt.Println(*number)
```
同时**new**关键字也可以给自定义类型分配内存：
```go
type Book struct {
   name string
   place int
}
var s *Book
s = new(Book) //分配空间
s.name ="百年孤独"
fmt.Println(s)
```
### make
我们来查看源码中官方给出的解释：
```go
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. The specification of the result depends on
// the type:
//	Slice: The size specifies the length. The capacity of the slice is
//	equal to its length. A second integer argument may be provided to
//	specify a different capacity; it must be no smaller than the
//	length. For example, make([]int, 0, 10) allocates an underlying array
//	of size 10 and returns a slice of length 0 and capacity 10 that is
//	backed by this underlying array.
//	Map: An empty map is allocated with enough space to hold the
//	specified number of elements. The size may be omitted, in which case
//	a small starting size is allocated.
//	Channel: The channel's buffer is initialized with the specified
//	buffer capacity. If zero, or the size is omitted, the channel is
//	unbuffered.
func make(t Type, size ...IntegerType) Type
```
>使内置函数分配和初始化类型对象切片、贴图或 chan（仅）。与 new 一样，第一个参数是类型，而不是价值。与 new 不同，make 的返回类型与其参数，而不是指向它指针。结果的规范取决于类型：切片：大小指定长度。切片的容量为等于它的长度。第二个整数参数可以提供给指定不同的容量;它必须小于长度。例如，make（int， 0， 10） 分配基础数组大小 10，并返回长度 0 和容量 10 的切片，这是由此基础数组支持。地图：为空地图分配了足够的空间来容纳指定的元素数。可以省略大小，在这种情况下分配一个小的起始大小。通道：通道的缓冲区使用指定的初始化缓冲区容量。如果为零，或省略大小，则通道为无缓冲。

**make**也是用来分配内存的，但是我们通过阅读官方手册可以知道，**make**只能用来给 chan、map 以及 slice来分配内存，并且他的返回值为类型的本身而不是指针

### 区别
	new可以给所有类型分配内存空间，而make只能给特定的几个类型分配
	new的返回值是该类型的指针而make的返回值是类型的本身

### 总结
在go语言中，make 关键字的主要作用是创建 slice、map 和 Channel 等内置的数据结构，而 new 的主要作用是为类型申请一片内存空间，并返回指向这片内存的指针。