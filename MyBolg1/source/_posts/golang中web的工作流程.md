---
title: golang中web的工作流程
tags:
  - golang
categories: golang
index_img: /img/webliuc.jpg
banner_img: /img/webliuc.jpg
abbrlink: 40086
date: 2020-11-09 15:23:11
---

# golang中web的工作流程

### 简要

首先我们来先看一下web的工作流程，下面是一张流程图

![](https://gitee.com/coderth/blogimage/raw/master/img/20201109153510.png)

### golang中web的工作流程

![](https://gitee.com/coderth/blogimage/raw/master/img/20201109153714.png)



这一张图是golang 的web工作流程，简单进行介绍一下：

* SOCKET：这个地方Web Server将绑定套接字（可以简单的把它理解成一个唯一id）
* BIND：这里进行端口号的绑定
* LISTEN：进行端口的监听
* LISTENT SOCKET：这个地方web server将阻塞并等待用户的请求
* CLIENT SOCKET：这里是WEB SERVER的处理中心，用户的所有请求都将在这里处理
* HANDLE REQUEST：这里相当于是一个处理函数（例如：请求头的读取，POST请求时请求体的读取；响应等）。

### 工作流程的模拟

当Web Server为Listen状态后，用户经过三次握手后与服务器建立连接，这个时候用户发送请求，该请求会发送到Client Socket处理中心去，处理中心调用处理函数，通过处理函数读取http请求的请求头以及请求体，再进行请求相关的操作，然后将响应的东西返回给处理中心，处理中心再返回给客户端。

### golang中启动http的方式

```go
package main

import (
	"fmt"
	"net/http"
)

type DefineServerMux struct{}


func (dsm *DefineServerMux) ServeHTTP(w http.ResponseWriter,r *http.Request){
	fmt.Fprintln(w,"创建自定义的多路复用处理器defineServerMux");
}

func ServeHTTP(w http.ResponseWriter, r *http.Request){
	fmt.Fprintln(w,"HandleFunc将一个普通的函数ServeHTTP包装Handle")
}
func main() {
	defineServerMux :=DefineServerMux{}
	//第一个参数是当前请求的路由，第二个参数是我们请求的事件
	http.Handle("/defineServerMux",&defineServerMux)
	//第一个参数是当前请求的路由，第二个参数是我们处理事件的方法
	http.HandleFunc("/",ServeHTTP)
	//绑定8080
	http.ListenAndServe(":8080",nil)
}


```

这是golang中启动http服务的方式，运行这段代码后，我们在浏览器中输入： http://127.0.0.1:8080/defineServerMux和http://127.0.0.1:8080/，结果如下

![](https://gitee.com/coderth/blogimage/raw/master/img/20201109160919.png)

![](https://gitee.com/coderth/blogimage/raw/master/img/20201109162549.png)

