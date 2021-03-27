---
title: RabbitMQ工作模式---Simple模式
tags:
  - RabbitMQ
  - 队列，高并发
categories: RabbitMQ
index_img: /img/RabbitMQSimple.jpg
banner_img: /img/RabbitMQSimple.jpg
abbrlink: 23073
date: 2020-10-17 15:04:58
---

# RabbitMQ工作模式---Simple模式

### 简介

Simple模式是RabbitMQ工作模式中对简单常用的模式，但是也是最基础的模式，所以我们需要充分的理解Simple工作模式，以下是Simple模式的架构图：

![](https://gitee.com/coderth/blogimage/raw/master/img/20201017151058.png)

P ：生产者

C：消费者



### 实现前的准备（省略了环境的安装）

1.创建一个Virtual host

![](https://gitee.com/coderth/blogimage/raw/master/img/20201017151540.png)

2.创建一个用户并将指定到刚才创建imooc的权限

![](https://gitee.com/coderth/blogimage/raw/master/img/20201017151338.png)





### 代码实现

1. 核心代码

   ```go
   package RabbitMQ
   
   import (
   	"fmt"
   	"github.com/streadway/amqp"
   	"log"
   )
   
   //url 格式 qmqp://账号：密码@rabbitmq服务器地址：端口号/vhost
   const MQURL = "amqp://imoocuser:imoocuser@127.0.0.1:5672/imooc"
   
   type RabbitMQ struct {
   	conn *amqp.Connection
   	channel *amqp.Channel
   	QueueName string //队列名称
   	Exchange string //交换机
   	Key string //key
   	Mqurl string //链接信息
   }
   
   
   //创建RabbitMQ结构体实例
   func NewRabbitMQ(queueName string,exchange string, key string) *RabbitMQ {
   	rabbitmq:= &RabbitMQ{QueueName: queueName,Exchange: exchange,Key: key,Mqurl: MQURL}
   	var err error
   	//创建rabbitmq链接
   	rabbitmq.conn,err = amqp.Dial(rabbitmq.Mqurl)
   	rabbitmq.failOnErr(err,"创建链接错误")
   	rabbitmq.channel,err = rabbitmq.conn.Channel()
   	rabbitmq.failOnErr(err,"获取channel失败")
   	return rabbitmq
   }
   
   
   //断开channel和connection
   func (r *RabbitMQ) Destory()  {
   	r.channel.Close()
   	r.conn.Close()
   
   }
   
   //错误处理函数
   func(r *RabbitMQ) failOnErr(err error,message string){
   	if err != nil {
   		log.Fatalf("%s:%s",err,message)
   		panic(fmt.Sprint("%s:%s",err,message))
   	}
   }
   //创建简单模式下的RabbitMQ
   //简单模式step：1。创建简单模式下的RabbitMQ
   func NewRabbitMQSimple(queueName string ) *RabbitMQ  {
   	return NewRabbitMQ(queueName,"","")
   }
   //简单模式step：2。简单模式下生产代码
   func (r *RabbitMQ) PublishSimple(message string)  {
   	//1.申请队列。如果队列不存在会自动创建，如果存在则跳过创建
   	//保证队列存在，消息能发送到队列；中
   	_,err:=r.channel.QueueDeclare(
   		r.QueueName,
   		//控制消息是否持久化
   		false,
   		//是否为自动删除
   		false,
   		//是否具有排他性
   		false,
   		//是否阻塞
   		false,
   		nil,
   		)
   	if err != nil {
   		fmt.Println(err)
   	}
   	//2.发送消息到队列中
   	r.channel.Publish(
   		r.Exchange,
   		r.QueueName,
   		//如果为true，更具exchange类型和routkey规则，如果无法找到符合条件的队列那么会把发送的小时返还给发送者
   		false,
   		//如果为true,当exchange发送消息到队列后发现队列上没有绑定消费者，则会把消息返还给发送着
   		false,
   		//
   		amqp.Publishing{
   			ContentType: "text/plain",
   			Body: []byte(message),
   		})
   }
   
   func (r *RabbitMQ) ConsumeSimple()  {
   	//1.申请队列。如果队列不存在会自动创建，如果存在则跳过创建
   	//保证队列存在，消息能发送到队列；中
   	_,err:=r.channel.QueueDeclare(
   		r.QueueName,
   		//控制消息是否持久化
   		false,
   		//是否为自动删除
   		false,
   		//是否具有排他性
   		false,
   		//是否阻塞
   		false,
   		nil,
   	)
   	if err != nil {
   		fmt.Println(err)
   	}
   	//2.接受消息
   	msgs,err:=r.channel.Consume(
   		r.QueueName,
   		//用来区分多个消费者
   		"",
   		//是否自动应答
   		true,
   		//是否具有排他性
   		false,
   		//如果设置为true，表示不能将同一个connection中发送的消息传递给这个connection中的消费者
   		false,
   		//队列是否阻塞
   		false,
   		nil,
   
   		)
   
   	if err != nil {
   		fmt.Println(err)
   	}
   	forever :=make(chan bool)
   	//3.启用携程处理消息
   	go func() {
   		for d :=range msgs {
   			//实现我们要处理的逻辑函数
   			log.Printf("Received a message:%s",d.Body)
   		}
   	}()
   	log.Println("[*] Waiting for messages,To exit press CTRL +C")
   	<-forever
   }
   ```

   

   2. 生产者使用方法

      ```go
      package main
      
      import (
      	"Imooc_Rabbitmq/RabbitMQ"
      	"fmt"
      )
      
      func main() {
      	rabbitmq := RabbitMQ.NewRabbitMQSimple(
      		"imoocSimple")
      	rabbitmq.PublishSimple("Hello imooc!")
      	fmt.Println("发送成功")
      }
      
      ```

      

   3. 消费者代码

      ```go
      package main
      
      import "Imooc_Rabbitmq/RabbitMQ"
      
      func main() {
      	rabbitmq :=RabbitMQ.NewRabbitMQSimple("imoocSimple")
      	rabbitmq.ConsumeSimple()
      }
      ```

      

### 运行结果

1. 首先启动消费者代码

   ![](https://gitee.com/coderth/blogimage/raw/master/img/20201017154407.png)

2. 启动生产者代码

   ![](https://gitee.com/coderth/blogimage/raw/master/img/20201017154455.png)

3. 这个时候查看消费者控制台

   ![](https://gitee.com/coderth/blogimage/raw/master/img/20201017154531.png)

4. 我们也可以在RabbitMQ中看到当前队列的状态

   ![](https://gitee.com/coderth/blogimage/raw/master/img/20201017154704.png)

