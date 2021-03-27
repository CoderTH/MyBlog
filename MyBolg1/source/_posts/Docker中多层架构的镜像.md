---
title: Docker中多层架构的镜像
tags: docker
categories: docker
index_img: /img/dockerASD.jpg
banner_img: /img/dockerASD.jpg
abbrlink: 9901
date: 2020-10-13 14:19:05
---

# Docker中多层架构的镜像

### 前言

Docker最值得称赞的一点就是他的方便性，我们运行一个应用就像拉取镜像并运行容器这么简单。不用担心安装或者他的依赖问题。但是随着docker的不断发展，事情开始变得越来越复杂，主要原因是添加了许多的新平台，比如windows，ARM等，我们会发现，在拉取镜像运行时，我们需要考虑镜像是否与当前的环境架构相匹配，这一点破坏了docker的流畅性，多架构镜像的出现解决了这个问题。



### 多架构镜像

现在docker规范已经支持了多架构镜像。这意味着某一个仓库标签下的镜像可以同时支持多种架构。简单的来说，就是一个镜像标签之下可以支持多个平台架构。为了实现这个特性，镜像仓库服务API支持两种重要的结构：Manifest列表和Manifest。Manifest列表是指墨一个镜像标签支持的架构列表。其支持每中架构，都有自己的Mainfest定义，其中列举了该镜像的构成

![](https://gitee.com/coderth/blogimage/raw/master/img/20201013143407.png)

图中使用了Golang光房镜像作为实例，左边是Manifest列表，其中包含了该镜像支持的每中架构。Manifest列表的每一项都有一个箭头，只想具体的Manifest，其中包含了镜像配置和镜像层数据

### 总结

创建支持多架构的镜像需要镜像的发布者做更多的工作。同时默写软件也并非跨平台的。在这个前提下，Manifest列表是可选的--在没有Manifest列表情况下，镜像仓库服务会返回普通的Manifest