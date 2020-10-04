---
title: 'golang中函数,方法和接口的浅析（函数篇）'
date: 2020-09-28 10:58:18
tags: [golang]
categories: golang
index_img: /img/qianxi.jpg
banner_img: /img/qianxi.jpg
---

# golang中函数，方法和接口的浅析（函数篇）

### 前言

函数对应操作序列，是程序的基本组成元素。Go语言中的函数有具名和匿名之分：具名函数一般对应于包级的函数，是匿名函数的一种特例。当匿名函数引用了外部的变量，那么这个匿名函数就变成了闭包函数，闭包函数是函数编程语言的核心。方法是绑定到一个具体类型的特殊函数，Go语言中的方法是依托于类型的，必须在编译时静态绑定。接口定义了方法的集合，这些方法依托于运行时的接口对象，因此接口对应的方法是在运行时动态绑定的。Go语言通过隐式接口机制实现了鸭子模型（duck typing）



### 函数

在golang中函数时第一类对象，可以将函数保存到变量中。函数主要有具名和匿名之分，包级函数一般都是具名函数，具名函数时匿名函数的一种特例。当然Go语言中每个类型还可以有自己的方法，方法其实也时函数的一种

```golang
//具名函数
func Add(a,b,int)int{
	return a+b
}
//匿名函数
var Add = func(a,b int) int{
	return a+b
}
```

Go语言中的函数可以有多个参数和多个返回值，参数和返回值都是以传值的反水和被调用者交换数据。在语法上，函数还支持可变数量的参数，可变数量的参数必须是最后出现的参数，可变数量的参数其实是一个切片类型的参数。

```goalng
//多个参数和多个返回值
func Swap(a,b int) (int,int){
	return b,a
}

//可变数量的参数
//more对应[]int切片类型
func Sum(a int,more ...int) int{
	for _, v:= range more{
		a +=v
	}
	retrun a
}
```



当可变参数是一个空接口时，调用者是否解包可变参数会导致不同的结果：

```golang
func maimn(){
	var a = []interface{}{123."abc"}
	Print(a...)//123 abc
	Print(a)//[123 abc]
}

Print(a ....interface{}){

	fmt.Println(a...)
}
```

第一个Print调用时传入的参数时a...,等价雨直接调用Print(123,"abc").第二个Println调用传入的时未解包的a，等价于直接调用Print([]interface{}{123,"abc"})。不仅函数的参数可以有名字，可以给函数的返回值命名：

````golang
func Find(m map[int]int,key int)(value int,ok bool){
		value,ok = m[key]
		return
}
````

如果返回值命名了，可以通过名字来修改返回值，也可以通过defer语句在return之后修改返回值

```golang
func Inc(v int){
	defer func(){v++}()
	return 42
}
```

其中defer语句延迟执行了一个匿名函数，应为这个函数不捕获了外部函数的局部变量v,这种函数我们一般称为闭包。闭包对捕获的外部变量并不是以传值方式访问，而实以引用方式访问。

闭包的这种以引用方式访问外部变量的行为可能会导致一些隐含的问题;

```golang
func main(){
	for i:= 0;i<3;i++{
		defer func() {println(i)}()
	}
}
```

输出：

> output
>
> 3
>
> 3
>
> 3

因为是闭包，在for迭代语句中，每一个defer语句延迟执行的函数引用的都是同一个i迭代变量，在循环结束后这个变量的值为3，因此最终输出的都是3.

修复的思路是在每轮迭代中为每个defer语句的闭包函数生成独有的变量。可以用下面两种方式：

```
func mian(){
	for i :=0;i<3;i++{
		i :=i//定义一个循环体内局部变量i、
		defer func(){println(i)}()
	}
}
-------------------------------------------------------
func main(){
		for i :=0;i<3;i++{
		//通过函数传入i
		//defer语句会马上对调用参数求值
		defer func(i int){println(i)}(i)
	}
}
```

第一种方法是在循环体内部再定义一个局部变量，这样每次迭代defer语句的闭包函数捕获的都是不同的变量，这些变量的值对应迭代时的值。第二种方式是将迭代变量通过闭包函数的参数传入，defer会马上对调用参数求值。两种方式都是可以工作的。不过一般来说，在for循环内部执行defer语句斌不是一个好的习惯。