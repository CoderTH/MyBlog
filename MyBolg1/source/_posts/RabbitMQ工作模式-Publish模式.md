---
title: RabbitMQ工作模式--Publish模式
date: 2020-10-21 08:25:40
tags: [RabbitMQ,队列，高并发]
categories: RabbitMQ
index_img: /img/publish.jpg
banner_img: /img/publish.jpg
---

# RabbitMQ工作模式--Publish模式



### 简介

消息被路由投递给多个队列，一个消息被多个消费者获取。下面是Publish模式的结构图

![](https://gitee.com/coderth/blogimage/raw/master/img/20201021083028.png)

从图中我们可以发现：1.在生产者和队列中间多了一个x，这个x代表的是交换机。2. 在publish中可以有多个队列，以及多个消费者。在Publish模式下，生产者生产的消息，能被多个消费者所获取。





### 代码实现

生产者实现：

```go
package main
 
import (
    "RabbitMQ"
    "strconv"
    "strings"
    "time"
)
 
func main(){
    ch := rabbitMQ.Connect("amqp://user:password@ip:port/")
    rabbitMQ.NewExchange("amqp://user:password@ip:port/","exchange1","fanout")
    i := 0
    for{
        time.Sleep(1)
        greetings :=  []string{"Helloworld!",strconv.Itoa(i)}
        ch.Publish("exchange1",strings.Join( greetings, " "),"")
        i = i+1
    }
 
}
```



### 消费者

```go
package main
 
import (
    rabbitMQ "RabbitMQ"
    "log"
)
 
func main(){
    // 1.接收者，首先创建自己队列
    // 2.创建交换机
    // 3.将自己绑定到交换机上
    // 4.接收交换机上发过来的消息
 
    //第一个参数指定rabbitmq服务器的链接，第二个参数指定创建队列的名字
    //1
    receive_mq := rabbitMQ.New("amqp://user:password@ip:port/","hello1")
    //2
    //第一个参数：rabbitmq服务器的链接，第二个参数：交换机名字，第三个参数：交换机类型
    rabbitMQ.NewExchange("amqp://user:password@ip:port/","exchange1","fanout")
    //3
    // 队列绑定到exchange
    receive_mq.Bind("exchange1","")
    //4
    for{
        //接收消息时，指定
        msgs := receive_mq .Consume()
        go func() {
            for d := range msgs {
                log.Printf("recevie1  Received a message: %s", d.Body)
            }
        }()
    }
}
```



### 消费者2

```go
package main
 
import (
    rabbitMQ "RabbitMQ"
    "log"
)
 
func main(){
    // 1.接收者，首先创建自己队列
    // 2.创建交换机
    // 3.将自己绑定到交换机上
    // 4.接收交换机上发过来的消息
 
    //第一个参数指定rabbitmq服务器的链接，第二个参数指定创建队列的名字
    //1
    receive_mq := rabbitMQ.New("amqp://user:password@ip:port/","hello2")
    //2
    //第一个参数：rabbitmq服务器的链接，第二个参数：交换机名字，第三个参数：交换机类型
    rabbitMQ.NewExchange("amqp://user:password@ip:port/","exchange1","fanout")
    //3
    // 队列绑定到exchange
    receive_mq.Bind("exchange1","")
    //4
    for{
        //接收消息时，指定
        msgs := receive_mq .Consume()
        go func() {
            for d := range msgs {
                log.Printf("recevie2  Received a message: %s", d.Body)
            }
        }()
    }
}
```



