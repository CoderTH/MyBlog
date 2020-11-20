---
title: go mod使用方法
date: 2020-11-20 09:22:49
tags: [golang]
categories: golang
index_img: /img/gomodel.jpg
banner_img: /img/gomodel.jpg
---

# go mod简单使用方法



### 前言

最近在使用goland学习的时候遇到需要使用第三方包的场景，以前都是直接使用go get 直接下载安装到Path下的src，但是在我更新了go版本后，代码并没有下载到src中而是到了同目录下的pkg/mod中，并且在项目中也无法应用，查了一些资料，发现是go modules的原因，这章主要讲一下go mod的简单使用方法，方便学习以复盘



### mac配置go mod环境变量出现的问题

其实在配置环境这个环境，我出了无数个错误，其中大部分都是因为mac下环境变量配置，我们先来看go mod需要配置那些环境变量

```go
export GO111MODULE=on
export GOPROXY=https://goproxy.io // 设置代理
```

第一行是开启mod的环境变量，第二行是设置代理，mac环境在～/.bash_profile中配置，然后使用source 让配置文件立即生效，但是我在使用了source后，当前窗口使用go env的确可以查看到变量已经生效，但是当我关闭窗口，重新运行窗口变量就失效了，查了一些资料说是，在mac下系统使用了zsh代替了bash，所以我在～/.zshrc最后一行中添加了source ~/.bash_profile.还是无果



### 解决方法

暂时没有找到很好的解决方法，所以我使用了一个非常笨的方法，就是每次当我使用go mod之前都去手动source 配置文件，或者直接运行下面两行代码，让环境变量临时生效：

```go
export GO111MODULE=on
export GOPROXY=https://goproxy.io // 设置代理
```



### goland中的配置

![](https://gitee.com/coderth/blogimage/raw/master/img/20201120095712.png)





### 案例

当我们配置好环境变量，那么我们就可以运行一个小案例了

* 第一步，我们使用goland创建一个modules项目

  这时候我们会发现，陌路下多了一个go mod文件（这个文件就是存放你引用三方包的版本）

* 第二部，创建main.go文件，如下代码：

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run() // listen and serve on 0.0.0.0:8080
}
```

在写这段代码的时候，你会发现goland会一直给你报红色，不用担心，这个时候直接运行代码，会帮你自动下载依赖包：

![](https://gitee.com/coderth/blogimage/raw/master/img/20201120100210.png)

这个时候代码就有自动补全了，并且下载下来的依赖包存储在以下目录：

![](https://gitee.com/coderth/blogimage/raw/master/img/20201120100307.png)

### 结语

如果有人知道环境变量的解决方法请联系我`CoderTh@outlook.com`，感激不尽！