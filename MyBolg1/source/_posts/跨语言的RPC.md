---
title: 跨语言的RPC
date: 2020-10-12 09:00:39
tags: golang
categories: golang
index_img: /img/kuayuyan.jpg
banner_img: /img/kuayuyan.jpg
---

# 跨语言的RPC

### 前言

标准库中的RPC默认采用GO语言特有的Gob编码，所有从其他语言调用Go语言实现的RPC服务将比较困难。在互联网的微服务时代，每一个RPC以及服务的使用者都可能采用不同的编程语言，因此跨语言是互联网时代RPC的一个首要条件。得益于RPC框架设计，Go语言的RPC其实也是很容易实现跨语言支持的。

Go语言的RPC框架有两个比较有特色的设计：第一个是RPC数据打包时可以通过插件实现自定义的编码和解码；另一个就是ROC建立在抽象的io.ReadWriteCloser接口之上，我们可以将RPC架设在不同的通信协议上。这里我们通过官方自带的jsonrpc扩展实现一个跨语言的小案例

### 案例



首先是基于JSON编码的重新实现

```go
package main

import (
	"log"
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
)

type HelloService struct {

}

func (p *HelloService) Hello(requset string,reply *string)error  {
	*reply = "hello:"+requset
	return nil

}

func main() {
	rpc.RegisterName("HelloService",new(HelloService))
	listener,err :=net.Listen("tcp",":1234")
	if err != nil {
		log.Fatal("ListenTCP error",err)
	}
	for  {
		conn,err:=listener.Accept()
		if err != nil {
			log.Fatal("Accept error",err)
		}
		go rpc.ServeCodec(jsonrpc.NewServerCodec(conn))
	}
}
```

代码和我们之前最大的区别就是用rpc.ServeCodec()代替了rpc.ServeConn()函数，传入的参数是针对服务器端的JSON编解码器。

然后时实现JSON版本的客户端：

```go
package main

import (
	"fmt"
	"log"
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
)

func main() {
	conn,err:=net.Dial("tcp","localhost:1234")
	if err != nil {
		log.Fatal("net Dial",err)
	}
	client:=rpc.NewClientWithCodec(jsonrpc.NewClientCodec(conn))
	var reply string
	client.Call("HelloService.Hello","hello",&reply)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(reply)
}

```

先使用net.Dial()函数建立TCP链接，然后基于该链接建立针对客户端的JSON编解码器。

在确保客户端可以正常调用RPC服务的方法之后，我们用一个普通的TCP服务代替Go 语言的RPC服务，我门先关闭前面我们编写的服务器端的代码，然后在命令行中输入 nc -l  1234启动一个相同端口号的tcp服务，然后再执行一次客户端的代码，这时我们就可以在nc中看到以下信息

![](https://gitee.com/coderth/blogimage/raw/master/img/20201012102158.png)

```go
{"method":"HelloService.Hello","params":["hello"],"id":0}
```

这是一个JSON编写的数据，其中method部分对应要调用的RPC服务和方法组合成的名字，params部分的第一个元素为参数，id是由调用放维护的唯一的调用编号

请求的JSON数据对象在内部其实对应了两个结构体，他们的结果基本相同，客户端是clientRequest,服务器端是serverRequest。

```go
type	clientRequest struct{
  Method string			'json:"method"'
  Params [1]interface{} 'json:"params"'
  Id			uint64				'json:id'
}

type	serverRequest struct{
  Method string			'json:"method"'
  Params [1]interface{} 'json:"params"'
  Id			uint64				'json:id'
}
```

在获取到RPC调用对应的JSON数据后，可以通过直接向架设了RPC服务的TCP服务器发送JSON数据模拟RPC方法调用

```go
echo -e '{"method":"HelloService.Hello","params":["hello"],"id":1}'|nc lacalhost 1234
```

返回如下

```go
{"id":1,"result":"hello:hello","error":null}
```

其中id对应输入的id参数，result为返回的结果，error部分在出问题时表示错误信息。对顺序调用来说，id不是必须的。但是GO语言的RPC框架支持一步调用，当返回结果的顺序和调用的顺序不一样时，可以通过id来识别对应的调用。





### 总结

经过上面的学习，我们知道，无论采用任何的语言，只要遵循了相同的JSON的结构，以同样的流畅就可以和GO语言编写RPC服务进行通信，这样就实现了跨语言的RPC。