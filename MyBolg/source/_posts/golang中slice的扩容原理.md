---
title: golang中slice的扩容原理
date: 2020-09-19 17:05:15
tags: [golang]
categories: golang
index_img: /img/golangzhong.jpg
banner_img: /img/golangzhong.jpg
---

# golang中slice的扩容

### 前言
学习golang肯定使用过slice（切片），大家都知道它是一种数据结构，这种数据结构便于使用和管理数据集合，并且他是围绕动态数组的概念构建的，可以按照自己的需求自动缩小或者扩容，但是在学习这个知识点的时候我并没有深究其扩容的底层原理，故对此写下这篇博客进行记录，方便我的学习以及后期的复盘

### 简介
切片的动态增长是通过内置函数append来实现的，这个函数可以快速高效的增长切片，还可以通过对切片再次切片来缩小一个切片的大小。切片是一个非常小的对象，它是对底层的数组进行了抽象，并且提供了相关的操作方法。他拥有三个字段，分别为：指向底层数组的指针，长度，容量。

### 内部的实现

![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jb2RlcnRoLmNuL3dwLWNvbnRlbnQvdXBsb2Fkcy8yMDIwLzA5LzVmNTVkZDc1YmI4MTMucG5n?x-oss-process=image/format,png)

### 切片的增长
相对于数组而言，使用切片的其中一个好处就是，切片可以按照自己的需求进行扩容，其中主要依赖的appen函数，他会处理增加长度的所有操作细节。
> append有两个参数，第一个是需要一个被操作的切片，第二个是你要追加的值。对于append追加值有两种情况，第一种就是扩容的范围在容量内，第二个是大小超过容量的大小，

下面这段代码为第一种情况：
```go
//创建一个整型切片
//长度和容量都为5
slice := []int{10,20,30,40,50}
//创建一个新的切片
//长度为两个元素，容量为四个元素
newSlice :=slice[1:3]

//使用原有的容量来分配一个新的元素
//将新的元素赋值为60
newSlice = append(newSlice,60)

```
上述代码中的append操作完以后，两个切片和底层的布局如图：
![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jb2RlcnRoLmNuL3dwLWNvbnRlbnQvdXBsb2Fkcy8yMDIwLzA5LzVmNTVmMmI4YzI4YTkucG5n?x-oss-process=image/format,png)
因为newSlice在底层数组里面还有额外的容量可用，append操作会将可用的元素合并到切片的长度中，并对其进行赋值。由于和原始的slice共享同一个底层数组，slice中索引为3的元素值也被改动了。如果底层的数组容量不够时，append函数会创建一个新的底层数组，将被引用的现有的值赋值进新的数组中，再追加新的值，如下代码：
```go
//创建一个整型切片
//其长度和容量都是4个元素
slice :=int{10,20,30,40}
//向切片追加一个新的元素
//将新元素设置为50
NewSlice :=append(slice,50)
```
### 总结
在golang中使用append来给slice扩容会出现两种情况，第一种是slice的容量足够时，会将新值直接插入到slice的底层数组中；第二种是slice的容量不够时，append操作会将slice的地址指针指向一个新的底层数组，将原来数组的数据赋值进新的数组中，再将新值插入，并且新的数组的容量为原来的两倍