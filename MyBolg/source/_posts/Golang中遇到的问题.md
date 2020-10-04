---
title: Golang中遇到的问题
date: 2020-09-28 10:18:45
tags: [golang]
categories: golang
index_img: /img/golangcuowu.jpg
banner_img: /img/golangcuowu.jpg
---

# Golang中常见的问题(持续更新)

### 前言

这里列举一些我们学习golang时常遇见的问题，他们都是符合golang语言语法的，可以正常的编译，但是可能会出现运行结果错误，或者是由资源泄露的风险

### 可变参数是空接口类型

当参数的可变参数是空接口类型时，传入空接口的切片时需要注意参数的展开的问题：

```golang
func main(){
	var a = []interface{}{1,2,3}
	fmt.Println(a)
	fmt.Println(a...)
}
```

不管参数是否展开，编译器都无法发现错误，但是输出是不同的：

> [1  2  3]
>
> 1 2 3

### 数组是值传递

在函数调用参数中，数组是值传递，无法通过修改数组类型的参数返回结果：

```golang
func main(){
	x := [3]int{1,2,3}
	func(arr [3]int){
		arr[0] = 7
		fmt.Println(arr)
	}(x)
	fmt.Println(x)
}
```

必要时需要使用切片。

### map遍历时顺序不固定

map是一种散列表实现，每次遍历的顺序都可能不一样：

```golang
func main(){
		m := map[string]string{
			"1" : "1",
			"2" : "2",
			"3" : "3",
		}
		for k,value:=range m {
			println(k,v)
		}
}
```

### 返回值被屏蔽

在局部变量中，命名的返回值被同名的局部变量屏蔽：

```
func Foo()(err error){
	if err : bar(); err != nill{
		return
	}
	return
}
```

### 结尾

本章博客将会一直更新.......