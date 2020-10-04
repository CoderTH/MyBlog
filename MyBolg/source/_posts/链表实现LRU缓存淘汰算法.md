---
title: 链表实现LRU缓存淘汰算法
date: 2020-10-03 14:04:08
tags: [golang, 算法]
categories: 算法
index_img: /img/LRUshixian.jpg
banner_img: /img/LRUshixian.jpg
---

# 链表实现LRU缓存淘汰算法

### 缓存

在了解LRU缓存淘汰算法之前我们需要先了解什么是缓存，其实缓存就是一种提高数据读取性能的技术，在硬件设计，软件开发中都有非常广泛的应用，比如常见的cpu缓存，数据库缓存，浏览器缓存等等。



### 什么是LRU缓存淘汰算法

首先我们需要知道缓存的大小是非常有限的，当缓存被占满时，有哪些数据应该清除，有哪些数据需要保存，这些都需要一些缓存策略来决定。最常见的三种方式：先进先出策略 FIFO（First In，First Out）、最少使用策略 LFU（Least Frequently Used）、最近最少使用策略 LRU（Least Recently Used）。

> 其实三种淘汰策略并不需要死记硬背，打个比方：假如说，你买了很多本技术书，但有一天你发现，这些书太多了，太占书房空间了，你要做个大扫除，扔掉一些书籍。那这个时候，你会选择扔掉哪些书呢？对应一下，你的选择标准是不是和上面的三种策略神似呢



### 如何实现LRU缓存淘汰算法

我这里暂时只进行思路的讲解：

我们可以维护一个有序的单链表，越靠近链表尾部的节点是越早之前访问的。当有一个新的数据被访问时候，我们从链表头开始顺序遍历。

1. 如果此数据之前已经被缓存在链表中了，我们遍历得到这个数据对应的节点。并将其从原来的位置删除，然后再插入到链表的头部
2. 如果此数据没有在缓存链表中，又可以分为两种情况。
   * 如果此时缓存未满，则将此节点直接插入到链表的头部；
   * 如果此时缓存已满，则链表尾节点删除，将新的数据节点插入链表的头部

这样我们就已经实现了一个LRU缓存。因为不管缓存有没有满，我们都需要遍历一边链表，所以这种基于链表的实现思路，缓存访问的时间复杂度都是O（n）。



### 以下是使用golang实现LRU缓存淘汰算法

```go
type Node struct {
    Key int
    Value int
    pre *Node
    next *Node
}

type LRUCache struct {
    limit int
    HashMap map[int]*Node
    head *Node
    end *Node
}

func Constructor(capacity int) LRUCache{
    lruCache := LRUCache{limit:capacity}
    lruCache.HashMap = make(map[int]*Node, capacity)
    return lruCache
}

func (l *LRUCache) Get(key int) int {
    if v,ok:= l.HashMap[key];ok {
        l.refreshNode(v)
        return v.Value
    }else {
        return -1
    }
}

func (l *LRUCache) Put(key int, value int) {
    if v,ok := l.HashMap[key];!ok{
        if len(l.HashMap) >= l.limit{
            oldKey := l.removeNode(l.head)
            delete(l.HashMap, oldKey)
        }
        node := Node{Key:key, Value:value}
        l.addNode(&node)
        l.HashMap[key] = &node
    }else {
        v.Value = value
        l.refreshNode(v)
    }
}

func (l *LRUCache) refreshNode(node *Node){
    if node == l.end {
        return
    }
    l.removeNode(node)
    l.addNode(node)
}

func (l *LRUCache) removeNode(node *Node) int{
    if node == l.end  {
        l.end = l.end.pre
    }else if node == l.head {
        l.head = l.head.next
    }else {
        node.pre.next = node.next
        node.next.pre = node.pre
    }
    return node.Key
}

func (l *LRUCache) addNode(node *Node){
    if l.end != nil {
        l.end.next = node
        node.pre = l.end
        node.next = nil
    }
    l.end = node
    if l.head == nil {
        l.head = node
    }
}
```



### 总结

以上就是使用链表实现LRU缓存淘汰算法的全部内容，其实这并不是实现LRU的最优办法，因为缓存访问的时间复杂度永远都为O（n），实际上我们可以继续优化这个实现思路，比如引入散列表来记录每一个数据的位置，可以将时间复杂度降到O（1）