---
title: cache2go浅析
date: 2020-11-11 14:49:10
tags: [golang]
categories: golang
index_img: /img/cachago2.jpg
banner_img: /img/cachago2.jpg
---

# cache2go浅析

### 简介

这两天发现了一个非常适合新手学习的开源项目，简单的研究了一天，写篇博客记录一下，方便以后的学习与复盘



### cache2go简介

该项目是使用golang实现的一个并发安全并且拥有着超时清除机制的缓存裤，里面存储的数据格式是key-value格式的，有点类似于redis。这里只对一些核心的代码进行讲解。

### 源码

![](https://gitee.com/coderth/blogimage/raw/master/img/20201111145815.png)

这是该项目的目录结构，其中最核心的代码就是红色标出的三个文件，接下来我们对他们一一进行讲解

> Cacheitem.go

```go
/*
 * Simple caching library with expiration capabilities
 *     Copyright (c) 2013-2017, Christian Muehlhaeuser <muesli@gmail.com>
 *
 *   For license see LICENSE.txt
 */

package cache2go

import (
	"sync"
	"time"
)

// CacheItem is an individual cache item
// Parameter data contains the user-set value in the cache.
type CacheItem struct {
	sync.RWMutex//读写锁，为了保证并发安全
	// The item's key.
	key interface{}//缓存的key
	// The item's data.
	data interface{}//缓存的数据
	// How long will the item live in the cache when not being accessed/kept alive.
	lifeSpan time.Duration//生命周期，当没该缓存项不再被访问时，还能在缓存中存活的时间
	// Creation timestamp.
	createdOn time.Time//缓存项的创建时间
	// Last access timestamp.
	accessedOn time.Time//缓存项的最后一次访问的时间
	// How often the item was accessed.
	accessCount int64//缓存项被访问的次数
	// Callback method triggered right before removing the item from the cache
	aboutToExpire []func(key interface{})//被删除时的回掉函数（这个函数在缓存被删除前调用）
}

// NewCacheItem returns a newly created CacheItem.
// Parameter key is the item's cache-key.
// Parameter lifeSpan determines after which time period without an access the item
// will get removed from the cache.
// Parameter data is the item's value.
//这个函数返回一个新创建的ChacheItem的指针
//
func NewCacheItem(key interface{}, lifeSpan time.Duration, data interface{}) *CacheItem {
	t := time.Now()
	return &CacheItem{
		key:           key,
		lifeSpan:      lifeSpan,
		createdOn:     t,
		accessedOn:    t,
		accessCount:   0,
		aboutToExpire: nil,
		data:          data,
	}
}

// KeepAlive marks an item to be kept for another expireDuration period.
//这个函数来更新缓存项的最近访问时间（相当于访问了缓存项以后调用该函数）
func (item *CacheItem) KeepAlive() {
	item.Lock()//为保证并发安全，枷锁
	defer item.Unlock()//运行最后解锁
	item.accessedOn = time.Now()//将最后一次访问时间设置为当前时间
	item.accessCount++//更新缓存项访问次数
}

// LifeSpan returns this item's expiration duration.
//获取当前缓存项的生命周期
func (item *CacheItem) LifeSpan() time.Duration {
	// immutable
	return item.lifeSpan
}

// AccessedOn returns when this item was last accessed.
//获取当前缓存项的最后一次访问时间
func (item *CacheItem) AccessedOn() time.Time {
	item.RLock()//为了防止出现竞争状态，给当前缓存项枷锁
	defer item.RUnlock()//函数最后解锁
	return item.accessedOn
}

// CreatedOn returns when this item was added to the cache.
//获取当前缓存项的创建时间（因为这个创建时间并不会被改变，所以并不会出现竞争状态，所以没有必要枷锁）
func (item *CacheItem) CreatedOn() time.Time {
	// immutable
	return item.createdOn
}

// AccessCount returns how often this item has been accessed.
//获取当前缓存项的访问次数
func (item *CacheItem) AccessCount() int64 {
	item.RLock()//同理，如果在获取访问次数的同时有人访问了该缓存项，那么就会出项竞争问题，所以需要将该
				//缓存项枷锁
	defer item.RUnlock()
	return item.accessCount
}

// Key returns the key of this cached item.
//获取当前缓存项的key
func (item *CacheItem) Key() interface{} {
	// immutable
	return item.key
}

// Data returns the value of this cached item.
//获取当前缓存项的数据value
func (item *CacheItem) Data() interface{} {
	// immutable
	return item.data
}

// SetAboutToExpireCallback configures a callback, which will be called right
// before the item is about to be removed from the cache.
//这里设置删除缓存项的回调函数
func (item *CacheItem) SetAboutToExpireCallback(f func(interface{})) {
	if len(item.aboutToExpire) > 0 {//因为结构体中定义的回调函数是切片函数类型，所以通过他的长度判断
									//是否存在回调函数，如果存在就删除
		item.RemoveAboutToExpireCallback()
	}
	item.Lock()
	defer item.Unlock()
	item.aboutToExpire = append(item.aboutToExpire, f)//将传入的参数f函数加入到切片当中
}

// AddAboutToExpireCallback appends a new callback to the AboutToExpire queue
//添加删除缓存项的回调函数
func (item *CacheItem) AddAboutToExpireCallback(f func(interface{})) {
	item.Lock()
	defer item.Unlock()
	item.aboutToExpire = append(item.aboutToExpire, f)
}

// RemoveAboutToExpireCallback empties the about to expire callback queue
//删除所有回调函数
func (item *CacheItem) RemoveAboutToExpireCallback() {
	item.Lock()
	defer item.Unlock()
	item.aboutToExpire = nil
}


```

> Cacheable.go

```go
/*
 * Simple caching library with expiration capabilities
 *     Copyright (c) 2013-2017, Christian Muehlhaeuser <muesli@gmail.com>
 *
 *   For license see LICENSE.txt
 */

package cache2go

import (
	"log"
	"sort"
	"sync"
	"time"
)

// CacheTable is a table within the cache
type CacheTable struct {
	sync.RWMutex//读写锁

	// The table's name.
	name string//表的名称
	// All cached items.
	items map[interface{}]*CacheItem//所有的缓存项（由map储存）

	// Timer responsible for triggering cleanup.
	cleanupTimer *time.Timer //定时器（可以触发缓存清理的触发器）
	// Current timer duration.
	cleanupInterval time.Duration//清除缓存的一个时间（周期）

	// The logger used for this table.
	logger *log.Logger//缓存表的日志

	// Callback method triggered when trying to load a non-existing key.
	//当访问一个不存在的缓存项时的回调函数
	loadData func(key interface{}, args ...interface{}) *CacheItem
	// Callback method triggered when adding a new item to the cache.
	//当向表中添加缓存项的时的回调函数
	addedItem []func(item *CacheItem)
	// Callback method triggered before deleting an item from the cache.
	//删除缓存表中的缓存项的时的回调函数
	aboutToDeleteItem []func(item *CacheItem)
}

// Count returns how many items are currently stored in the cache.
//获取缓存表中缓存项的长度
func (table *CacheTable) Count() int {
	table.RLock()
	defer table.RUnlock()
	return len(table.items)
}

// Foreach all items
//遍历缓存项中的缓存数据
func (table *CacheTable) Foreach(trans func(key interface{}, item *CacheItem)) {
	table.RLock()
	defer table.RUnlock()

	for k, v := range table.items {
		trans(k, v)
	}
}

// SetDataLoader configures a data-loader callback, which will be called when
// trying to access a non-existing key. The key and 0...n additional arguments
// are passed to the callback function.
//设置当试图访问一个不存在缓存的数据时的回调函数
func (table *CacheTable) SetDataLoader(f func(interface{}, ...interface{}) *CacheItem) {
	table.Lock()
	defer table.Unlock()
	table.loadData = f
}

// SetAddedItemCallback configures a callback, which will be called every time
// a new item is added to the cache.
//设置添加缓存项的回调函数
func (table *CacheTable) SetAddedItemCallback(f func(*CacheItem)) {
	if len(table.addedItem) > 0 {
		table.RemoveAddedItemCallbacks()
	}
	table.Lock()
	defer table.Unlock()
	table.addedItem = append(table.addedItem, f)
}

//AddAddedItemCallback appends a new callback to the addedItem queue
func (table *CacheTable) AddAddedItemCallback(f func(*CacheItem)) {
	table.Lock()
	defer table.Unlock()
	table.addedItem = append(table.addedItem, f)
}

// RemoveAddedItemCallbacks empties the added item callback queue
func (table *CacheTable) RemoveAddedItemCallbacks() {
	table.Lock()
	defer table.Unlock()
	table.addedItem = nil
}

// SetAboutToDeleteItemCallback configures a callback, which will be called
// every time an item is about to be removed from the cache.
func (table *CacheTable) SetAboutToDeleteItemCallback(f func(*CacheItem)) {
	if len(table.aboutToDeleteItem) > 0 {
		table.RemoveAboutToDeleteItemCallback()
	}
	table.Lock()
	defer table.Unlock()
	table.aboutToDeleteItem = append(table.aboutToDeleteItem, f)
}

// AddAboutToDeleteItemCallback appends a new callback to the AboutToDeleteItem queue
func (table *CacheTable) AddAboutToDeleteItemCallback(f func(*CacheItem)) {
	table.Lock()
	defer table.Unlock()
	table.aboutToDeleteItem = append(table.aboutToDeleteItem, f)
}

// RemoveAboutToDeleteItemCallback empties the about to delete item callback queue
func (table *CacheTable) RemoveAboutToDeleteItemCallback() {
	table.Lock()
	defer table.Unlock()
	table.aboutToDeleteItem = nil
}

// SetLogger sets the logger to be used by this cache table.
func (table *CacheTable) SetLogger(logger *log.Logger) {
	table.Lock()
	defer table.Unlock()
	table.logger = logger
}

// Expiration check loop, triggered by a self-adjusting timer.
//由计时器触发的到期检查
func (table *CacheTable) expirationCheck() {
	table.Lock()//先给表进行枷锁
	if table.cleanupTimer != nil {
		table.cleanupTimer.Stop()
	}
	if table.cleanupInterval > 0 {
		table.log("Expiration check triggered after", table.cleanupInterval, "for table", table.name)
	} else {
		table.log("Expiration check installed for table", table.name)
	}

	// To be more accurate with timers, we would need to update 'now' on every
	// loop iteration. Not sure it's really efficient though.
	now := time.Now()
	smallestDuration := 0 * time.Second
	for key, item := range table.items {
		// Cache values so we don't keep blocking the mutex.
		item.RLock()
		lifeSpan := item.lifeSpan
		accessedOn := item.accessedOn
		item.RUnlock()

		if lifeSpan == 0 {
			continue
		}
		//如果最后一次访问时间到现在的时间超过了他的生命周期，就删除该项
		if now.Sub(accessedOn) >= lifeSpan {
			// Item has excessed its lifespan.
			table.deleteInternal(key)
		} else {
			// Find the item chronologically closest to its end-of-lifespan.
			//按时间顺序查找最接近其寿命结束的项目
			if smallestDuration == 0 || lifeSpan-now.Sub(accessedOn) < smallestDuration {
				smallestDuration = lifeSpan - now.Sub(accessedOn)
			}
		}
	}

	// Setup the interval for the next cleanup run.
	//为下一次的清除设置时间（根据当前表中最接近结束的项目）
	table.cleanupInterval = smallestDuration
	if smallestDuration > 0 {
		table.cleanupTimer = time.AfterFunc(smallestDuration, func() {
			go table.expirationCheck()
		})
	}
	table.Unlock()
}

func (table *CacheTable) addInternal(item *CacheItem) {
	// Careful: do not run this method unless the table-mutex is locked!
	// It will unlock it for the caller before running the callbacks and checks
	table.log("Adding item with key", item.key, "and lifespan of", item.lifeSpan, "to table", table.name)
	table.items[item.key] = item

	// Cache values so we don't keep blocking the mutex.
	expDur := table.cleanupInterval
	addedItem := table.addedItem
	table.Unlock()

	// Trigger callback after adding an item to cache.
	if addedItem != nil {
		for _, callback := range addedItem {
			callback(item)
		}
	}

	// If we haven't set up any expiration check timer or found a more imminent item.
	if item.lifeSpan > 0 && (expDur == 0 || item.lifeSpan < expDur) {
		table.expirationCheck()
	}
}

// Add adds a key/value pair to the cache.
// Parameter key is the item's cache-key.
// Parameter lifeSpan determines after which time period without an access the item
// will get removed from the cache.
// Parameter data is the item's value.
func (table *CacheTable) Add(key interface{}, lifeSpan time.Duration, data interface{}) *CacheItem {
	item := NewCacheItem(key, lifeSpan, data)

	// Add item to cache.
	table.Lock()
	table.addInternal(item)

	return item
}

func (table *CacheTable) deleteInternal(key interface{}) (*CacheItem, error) {
	r, ok := table.items[key]
	if !ok {
		return nil, ErrKeyNotFound
	}

	// Cache value so we don't keep blocking the mutex.
	aboutToDeleteItem := table.aboutToDeleteItem
	table.Unlock()

	// Trigger callbacks before deleting an item from cache.
	if aboutToDeleteItem != nil {
		for _, callback := range aboutToDeleteItem {
			callback(r)
		}
	}

	r.RLock()
	defer r.RUnlock()
	if r.aboutToExpire != nil {
		for _, callback := range r.aboutToExpire {
			callback(key)
		}
	}

	table.Lock()
	table.log("Deleting item with key", key, "created on", r.createdOn, "and hit", r.accessCount, "times from table", table.name)
	delete(table.items, key)

	return r, nil
}

// Delete an item from the cache.
func (table *CacheTable) Delete(key interface{}) (*CacheItem, error) {
	table.Lock()
	defer table.Unlock()

	return table.deleteInternal(key)
}

// Exists returns whether an item exists in the cache. Unlike the Value method
// Exists neither tries to fetch data via the loadData callback nor does it
// keep the item alive in the cache.
func (table *CacheTable) Exists(key interface{}) bool {
	table.RLock()
	defer table.RUnlock()
	_, ok := table.items[key]
	return ok
}

// NotFoundAdd checks whether an item is not yet cached. Unlike the Exists
// method this also adds data if the key could not be found.
//从缓存查找是否存在传入key的缓存项，如果存在返回false，如果不存在创建该缓存项
func (table *CacheTable) NotFoundAdd(key interface{}, lifeSpan time.Duration, data interface{}) bool {
	table.Lock()

	if _, ok := table.items[key]; ok {
		table.Unlock()
		return false
	}

	item := NewCacheItem(key, lifeSpan, data)
	table.addInternal(item)

	return true
}

// Value returns an item from the cache and marks it to be kept alive. You can
// pass additional arguments to your DataLoader callback function.
//从缓存中去访问缓存项，如果存在更新他的活性，如果不存在调用loadData函数，并创建
func (table *CacheTable) Value(key interface{}, args ...interface{}) (*CacheItem, error) {
	table.RLock()
	r, ok := table.items[key]
	loadData := table.loadData
	table.RUnlock()

	if ok {
		// Update access counter and timestamp.
		r.KeepAlive()
		return r, nil
	}

	// Item doesn't exist in cache. Try and fetch it with a data-loader.
	if loadData != nil {
		item := loadData(key, args...)
		if item != nil {
			table.Add(key, item.lifeSpan, item.data)
			return item, nil
		}

		return nil, ErrKeyNotFoundOrLoadable
	}

	return nil, ErrKeyNotFound
}

// Flush deletes all items from this cache table.
func (table *CacheTable) Flush() {
	table.Lock()
	defer table.Unlock()

	table.log("Flushing table", table.name)

	table.items = make(map[interface{}]*CacheItem)
	table.cleanupInterval = 0
	if table.cleanupTimer != nil {
		table.cleanupTimer.Stop()
	}
}

// CacheItemPair maps key to access counter
type CacheItemPair struct {
	Key         interface{}
	AccessCount int64
}

// CacheItemPairList is a slice of CacheIemPairs that implements sort.
// Interface to sort by AccessCount.
type CacheItemPairList []CacheItemPair

func (p CacheItemPairList) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }
func (p CacheItemPairList) Len() int           { return len(p) }
func (p CacheItemPairList) Less(i, j int) bool { return p[i].AccessCount > p[j].AccessCount }

// MostAccessed returns the most accessed items in this cache table
func (table *CacheTable) MostAccessed(count int64) []*CacheItem {
	table.RLock()
	defer table.RUnlock()

	p := make(CacheItemPairList, len(table.items))
	i := 0
	for k, v := range table.items {
		p[i] = CacheItemPair{k, v.accessCount}
		i++
	}
	sort.Sort(p)

	var r []*CacheItem
	c := int64(0)
	for _, v := range p {
		if c >= count {
			break
		}

		item, ok := table.items[v.Key]
		if ok {
			r = append(r, item)
		}
		c++
	}

	return r
}

// Internal logging method for convenience.
func (table *CacheTable) log(v ...interface{}) {
	if table.logger == nil {
		return
	}

	table.logger.Println(v...)
}

```

> Cache.go

```go
/*
 * Simple caching library with expiration capabilities
 *     Copyright (c) 2012, Radu Ioan Fericean
 *                   2013-2017, Christian Muehlhaeuser <muesli@gmail.com>
 *
 *   For license see LICENSE.txt
 */

package cache2go

import (
	"sync"
)

var (
	cache = make(map[string]*CacheTable)
	mutex sync.RWMutex
)

// Cache returns the existing cache table with given name or creates a new one
// if the table does not exist yet.
//返回现有的所有表的与名称，如果不存在就创建
func Cache(table string) *CacheTable {
	mutex.RLock()
	t, ok := cache[table]
	mutex.RUnlock()

	if !ok {
		mutex.Lock()
		t, ok = cache[table]
		// Double check whether the table exists or not.
		if !ok {
			t = &CacheTable{
				name:  table,
				items: make(map[interface{}]*CacheItem),
			}
			cache[table] = t
		}
		mutex.Unlock()
	}

	return t
}

```

### 实例

项目本身为我们提供了几个简单的实例，我们挑其中一个来展示：

> Mycachedapp.go

```go
package main

import (
	cache2go "cache2go-master"
	"fmt"
	"time"


)

// Keys & values in cache2go can be of arbitrary types, e.g. a struct.
type myStruct struct {
	text     string
	moreData []byte
}

func main() {
	// Accessing a new cache table for the first time will create it.
	//创建myCache表
	cache := cache2go.Cache("myCache")

	// We will put a new item in the cache. It will expire after
	// not being accessed via Value(key) for more than 5 seconds.
	//给表中添加somekey的数据
	val := myStruct{"This is a test!", []byte{}}
	cache.Add("someKey", 5*time.Second, &val)

	// Let's retrieve the item from the cache.
	res, err := cache.Value("someKey")
	if err == nil {
		fmt.Println("Found value in cache:", res.Data().(*myStruct).text)
	} else {
		fmt.Println("Error retrieving value from cache:", err)
	}

	// Wait for the item to expire in cache.
	//等待六秒让先前设置的缓存项失效
	time.Sleep(6 * time.Second)
	res, err = cache.Value("someKey")
	if err != nil {
		fmt.Println("Item is not cached (anymore).")
	}

	// Add another item that never expires.
	cache.Add("someKey", 0, &val)

	// cache2go supports a few handy callbacks and loading mechanisms.
	cache.SetAboutToDeleteItemCallback(func(e *cache2go.CacheItem) {
		fmt.Println("Deleting:", e.Key(), e.Data().(*myStruct).text, e.CreatedOn())
	})

	// Remove the item from the cache.
	cache.Delete("someKey")

	// And wipe the entire cache table.
	cache.Flush()
}

```

