---
title: Rabbit工作模式--work模式
tags:
  - RabbitMQ
  - 队列，高并发
categories: RabbitMQ
index_img: /img/work.jpg
banner_img: /img/work.jpg
abbrlink: 45015
date: 2020-10-19 09:39:55
---

# Rabbit工作模式--work模式

### Work，工作模式

一个消息只能被一个消费者获取，当生产者生产一个消息以后会发送到队列中，这个消息会被消费者获取，但是这个消息只能够由一个消费者所获取，这个消息只能被消费一次，不能被多个消费者获取

![](https://gitee.com/coderth/blogimage/raw/master/img/20201019094347.png)

### 代码实例

simple模式的代码这里就不赘述了（上一篇博客有）

首先看work模式中的生产者代码：

```go
package main

import (
	"Imooc_Rabbitmq/RabbitMQ"
	"fmt"
	"strconv"
	"time"
)

func main() {
	rabbitmq := RabbitMQ.NewRabbitMQSimple("" +
		"imoocSimple")
	for i:=0;i<=100;i++ {
		rabbitmq.PublishSimple("Hello imooc!"+strconv.Itoa(i))
		time.Sleep(1 * time.Second)
		fmt.Println(i)
	}
}

```



我们在生产者代码中首先声明了一个simple模式的实例，然后我们每隔一秒发送一条消息，为了区分我们在Hello imooc后面添加了一个编号。

我们来看消费者一的代码：

```go
package main

import "Imooc_Rabbitmq/RabbitMQ"

func main() {
	rabbitmq :=RabbitMQ.NewRabbitMQSimple(""+"imoocSimple")
	rabbitmq.ConsumeSimple()
}

```

消费者二的代码

```go
package main

import "Imooc_Rabbitmq/RabbitMQ"

func main() {
	rabbitmq :=RabbitMQ.NewRabbitMQSimple(""+"imoocSimple")
	rabbitmq.ConsumeSimple()
}

```

我们会发现两个重要的点

* work模式下的消费者和生产者代码和simple模式的一摸一样
* 区别于simple模式，work模式多了一个消费者



运行截图：

![](https://gitee.com/coderth/blogimage/raw/master/img/20201019095919.png)





![](https://gitee.com/coderth/blogimage/raw/master/img/20201019095944.png)

![](https://gitee.com/coderth/blogimage/raw/master/img/20201019100708.png)

对比两个消费者的运行截图我们很容易发现：他们的编号一个是基数一个是偶数，说明一条消息的确是只能被消费一次





### 总结

在work模式中，其实代码的编写都和simple模式一样，唯一区别simple 模式的就是，在sumple模式中只有一个消费者，而在work模式中可以有两个甚至多个消费者