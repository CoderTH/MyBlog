---
title: Golang中RPC版的Hello World
tags:
  - glang
categories: golang
index_img: /img/RPC.jpg
banner_img: /img/RPC.jpg
abbrlink: 21625
date: 2020-10-09 08:35:47
---

# Glang中RPC版的Hello World

### 什么是RPC

RPC是远程调用（Remote Procedure Call）的缩写，简单的来说就是调用远处的一个函数，这个距离有可能是同一个文件的函数，也有可能是同一个及其的另一个进程的函数，还有可能是更远地方的函数。



### RPC版的hello world

我们首先构造一个HelloService类型，其中的hello()的方法用于实现一个打印的功能

```go
type HelloService struct{}
func (p *HelloService)Hello(request string,reply *string) error{
  *reply = "hello:"+request
  return nill
}
```

Hello方法必须满足Go语言的RPC规则：方法只能有两个可序列化的参数，其中第二个参数是指针类型，并且返回一个error类型，同时必须是公开的方法。

然后就可以将helloService类型的对象注册为一个RPC服务

```go
func main(){
  rpc.RegisterName("HelloService",new(HelloService))
  listener,err :=net.Listen("tcp",":1234")
  if err != nill{
    log.Fatal("ListenTCP error:",err)
  }
  rpc.ServeConn(conn)
}
```

其中rpc.RegisterName()函数调用会将对象类型中所有满足RPC规则的对象方法都注册成为一个RPC函数，宾切使用Listen方法给函数绑定一个为一个TCP链接，并且通过  rpc.ServeConn(conn)函数在该TCP链接上为对方提供RPC服务

下面是客户端调用RPC函数的方法

```go
func main(){
  client,err :=rpc.Dial("tcp","localhost:1234")
  if err!=nil{
    Log.Fatal("dialing:",err)
  }
  var reply string
  err = client.Call("HelloService.Hello","hello",&reply)
  if err !=nil{
    Log.Fatal(err)
  }
  fmt.Println(replu)
}
```

首先是通过rpc.Dial拨号RPC服务，然后通过client.Call()调用具体的RPC方法。在调用client.Call()时，第一个参数使用点好链接的RPC服务名字和方法名字，第二个和第三个参数分别是定义RPC方法的两个参数



