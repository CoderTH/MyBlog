---
title: 使用defer解决实际问题
date: 2020-12-03 16:11:46
tags: [golang]
categories: golang
index_img: /img/defer.jpg
banner_img: /img/defer.jpg
---

# 使用defer解决实际问题



### 引入

首先我们以leetcode上面的一道题目引入

![](https://gitee.com/coderth/blogimage/raw/master/img/20201203161516.png)

### 思路

这道题非常的简单，我们简单的提取一下关键字：`头节点`，`从尾到头`。其实读到这里我们就有了做这道题的思路，链表的头节点说明我们的数据源在链表中，从尾到头其实就是栈规则，它遵循一个先进后出的原则，而这道题我们就可以使用这个思想来解题。我们首先按照正常思路来实现

### 方法一

```go
type stack []int

func (s *stack) push(n int) {
	*s = append(*s, n)
}
func (s *stack) pop() int {
	res := (*s)[len(*s)-1]
	*s = (*s)[:len(*s)-1]
	return res
}
func (s *stack) len() int {
	return len(*s)
}
func reversePrint01(head *ListNode) []int {
	stack := new(stack)
	for head != nil {
		stack.push(head.Val)
		head = head.Next
	}

	l := stack.len()
	res := make([]int, l)

	for i := 0; i < l; i++ {
		res[i] = stack.pop()
	}
	return res
}

```

 这种解题方式就是我们手动实现了一个栈规则，并将数据以栈的规则入栈并返回。讲到这里可能大家会觉得偏移了本章的主题，别着急，接下来就进入本章的主题了

### defer简介

相信大家多多少少都使用过defer这个关键字，用的比较多的场景：异常的捕获、更加安全的资源回收等，其实我们在开发中使用它时关注的点都是其功能的本身，并没有过多的在意其的运行机制，所以本章就先简单的讲解一下其中的机制



### 多个defer的运行顺序

在程序中如果我们使用多个defer他的运行顺序会是什么呢？我们来看一下下面这段代码：

```go
package main

import "fmt"

func main() {
	defer fmt.Println(1)
	defer fmt.Println(2)
	defer fmt.Println(3)
	defer fmt.Println(4)
}
```

![](https://gitee.com/coderth/blogimage/raw/master/img/20201203163250.png)

我们不需要很仔细的看这段代码就可以发现，其实defer的运行机制其实就是符合栈规则的，先进后出

### 参数的计算

我们再来砍断代码：

```go
func main() {
	for i := 0; i < 5; i++ {
		defer fmt.Println(i)
	}
}
```

这次我们并没有直接使用常量来进行输出，而是通过循环变量来输出，下面是运行截图

![](https://gitee.com/coderth/blogimage/raw/master/img/20201203163744.png)

其实在GO中所有的函数调用都是传值的，defer所软是一个关键字，但是他也继承了这个特性，如果我们想要计算main函数的运行时间，我们可能会写出下面的一段代码

```go
func main() {
	startedAt := time.Now()
	defer fmt.Println(time.Since(startedAt))
	
	time.Sleep(10*time.Second)
}
```

其实这段代码的运行结果是不符合我们的预期的，进过简单的分析其实我们可以发现在调用defer关键字时defer会立刻将参数拷贝，然后再进行延迟运行，最后会输出0秒：

![](https://gitee.com/coderth/blogimage/raw/master/img/20201203164339.png)

虽然这里输出的是265ns,但是他并没有输出我们sleep的时间，所以并没有实现我们计算mian函数运行时间的需求。

其实解决这个问题也非常的简单，我们只需要将计算时间这段业务代码放在匿名函数中就有可以解决：

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	startedAt := time.Now()
	defer func() { fmt.Println(time.Since(startedAt)) }()


	time.Sleep(10*time.Second)
}
```

![](https://gitee.com/coderth/blogimage/raw/master/img/20201203164658.png)

这是什么原理呢？虽然defer关键字使用值传递，但是应为拷贝的是函数指针，所以`time.Since(startedAt)`会在mian函数返回后调用并且返回我们想要的结果

### 回到题目

我们对defer的特性进行了一个简单的介绍，其实我们能发现，他拥有着栈的特性，所以我们可以结合他的特性来解这道题目：

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reversePrint(head *ListNode) (arry []int) {
    
    for head!=nil{
        tamp := head
    defer func(){
            arry = append(arry,tamp.Val)
        }()
        head = head.Next
    }
    return 
}
```

![](https://gitee.com/coderth/blogimage/raw/master/img/20201203165232.png)