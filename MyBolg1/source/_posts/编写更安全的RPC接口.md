---
title: 编写更安全的RPC接口
tags: golang
categories: golang
index_img: /img/rpcanquan.jpg
banner_img: /img/rpcanquan.jpg
abbrlink: 59926
date: 2020-10-11 14:11:05
---



# 编写更安全的RPC接口

### 前言

在一般的RPC应用当中，作为开发人员一般分为了三种，第一种就是提供RPC服务的开发人员，第二种就是客户端使用RPC服务的开发人员，以及最重要的设计RPC接口和规范RPC接口的开发人员，前面的案例当中我们将三种角色融在了一起，虽然看起来非常的方便，但是非常的不利于后期的维护以及二次开发



### RPC接口规范

如果要冲高HelloService服务，第一步需要明确服务的名字以及接口（HelloService服务在上两篇博客）



```go
const HelloServiceName="path/to/pkg.helloservice"
type HelloServiceInterFace = interface{
  Hello(request string,reply *string)error
}
func RegisterHelloService(svc HelloServiceinterface)error{
  return rpc.RegisterName(HelloServiceName,svc)
}
```

我们把RPC服务的接口规范分为三个部分：首先是服务的名称，然后时服务要实现的详细方法列表，最后时注册该类服务的函数。为了避免名字的冲突，我们在PRC无法的名字中增加了包路径前缀（这个是RPC服务抽象的包路径，并非完全等价于Go语言的包路径）。RegisterHelloService注册服务的时候，编译器会要求唇乳的对象满足HelloServiceInterface接口。

在定义了ROC服务接口的规范以后，客户端就可以更具规范编写RPC调用的接口了

```go
func main(){
  client , err:=rpc.Dial("tcp","localhost:1234")
  if err !=nil{
    log.Fatal("dialing:",err)
  }
  var reply string
  err = client.Call(HelloServiceName+".Hello","hello",&reply)
  if err !=nil{
    log.Fatal(err)
  }
}
```

其中唯一的变化是client.Call()的第一个参数用HelloServiceName+".Hello"代替了“HelloService.Hello"然而通过client.Call()函数调用RPC方法依然比较烦碎，同时参数的类型依然无法得到编译器提供的安全保障

为了简化客户端用户调用RPC函数，我们可以在接口规范部分增加对库护短的简单包装：



```go
type HelloServiceClient struct{
  *rpc.Client
}
var _ HelloServiceInterface = (*HelloServiceClient)(nil)
func DialHelloService(network,address string)(*HelloServiceClient,error){
  c,err :=rpc.Dial(network,address)
  if err !=nil{
    return nil,err
  }
  return &HelloServiceClient{Client:c},nil
}

func (p *HelloServiceClient) Hello(request string,reply *string)error{
  return p.Client.Call(HelloServiceName+".Hello",request,reply)
}
```

我们在接口规范中针对客户新增加了HelloServiceClient类型，该类型也必须满足HelloServiceInterface接口，这样客户端用户就可以直接通过接口的方法调用RPC函数。

```go
func main(){
  client,err :=DialHelloService("tcp,"lacalhost:1234")
if err!=nil{
  log.Fatal("dialing:",err)
 }
 var reply string
err = client.Hello("hello",&repluy)
if err!=nil{
     log.Fatal(err)
	}
}
```

现在客户端用户不用再担心RPC方法名称或参数类型不匹配等低级错误的发生

