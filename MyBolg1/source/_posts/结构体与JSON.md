---
title: 结构体与JSON
date: 2020-10-03 12:05:04
tags: [golang, 基础]
categories: golang
index_img: /img/jiegoutiyujson.jpg
banner_img: /img/jiegoutiyujson.jpg
---

# golang中结构体与json

### 简介

在进行前后分离式开发时，json显得格外的重要，因为他是链接前后台重要的枢纽，在go语言中没有显式的对象，我们通常使用结构体来实现面向对象编程，本篇主要讲的是golang中结构体与json·的序列化以及反序列化。



### 什么是json

json是储存和交换文本信息的语法，他类似于xml，但是他比xml更加的便捷，快速，易于解析。主要使用场景就是作为前后台数据交互的枢纽，以下是一个简单json的格式：

```json
{
    "name":"huweicheng",
    "age":99
}
```



### 将结构体序列化

在golang中有非常丰富的库，我们可以直接使用"encoding/json"包中的Marshal函数进行结构体的序列化，以下是案例：

```go
package main

import (
	"encoding/json"
	"fmt"
)

//结构体与json
//1，把go中的结构体变量--->json字符串
//2。把json字符串--->go语言中能够识别的结构体
type persion struct {
	Name string `json:"name"`
	Age int`json:"age"`
}
func main() {
	p1:=persion{
		Name: "huweicheng",
		Age: 99,
	}
	//将p1转json
	b,err:=json.Marshal(p1)//序列号
	if err!=nil {
		fmt.Printf("marshal fail,err:%v",err)
		return
	}
	fmt.Printf(string(b))
	
}
```



运行结果

> {"name":"huweicheng","age":99}

在使用这个Marshal时有几个需要特别注意的地方：在申明结构体时字段的首写字母必须大写，这里牵扯到一个字段可见性的问题，小写代表了私有的，只能在当前包中使用，而使用Marshal时如果字段为小写那么将无法访问到字段；一般来说前台接受到的字段名称是小写的，如果将字段名称设置为大写，那么返回给前台的key也将是大写的，我们可以在申明字段的时候在后面加一个tag，作用类似于别名，写法如下：

```go
type persion struct {
	Name string `json:"name"`
	Age int`json:"age"`
}
```



### 将json反序列化

例子如下：

```go
package main

import (
	"encoding/json"
	"fmt"
)

//结构体与json
//1，把go中的结构体变量--->json字符串
//2。把json字符串--->go语言中能够识别的结构体
type persion struct {
	Name string `json:"name"`
	Age int`json:"age"`
}
func main() {
	p1:=persion{
		Name: "huweicheng",
		Age: 99,
	}
	//将p1转json
	b,err:=json.Marshal(p1)//序列号
	if err!=nil {
		fmt.Printf("marshal fail,err:%v",err)
		return
	}
	fmt.Printf(string(b))
	//反序列话
	str := `{"name":"huweicheng","age":99}`
	var p2 persion
	json.Unmarshal([]byte(str),&p2)//传指针是为了能在函数内部修改p2
	fmt.Printf("%#v\n",p2)
}

```

运行结果：



> {"name":"huweicheng","age":99}main.persion{Name:"huweicheng", Age:99}



