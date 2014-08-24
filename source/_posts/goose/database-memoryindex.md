title: goose自己写的搜索引擎-内存索引database.MemoryIndex
date: 2014-08-24 20:06:31
categories: 
- 技术
- 搜索引擎
tags:
- goose
- golang
- 搜索引擎
---

MemoryIndex模块用于管理内存倒排索引,类似于DiskIndex,但是简单很多.

<!-- more -->

##一、设计思想
MemoryIndex在实现的时候有这样的一些规则：

* 纯内存,所持有的内存如果不能及时保存到磁盘,一旦程序崩溃,全部数据丢失.
* 读操作并发下安全,以保证并发检索不会互相干扰.
* 写操作不支持并发,而且写操作会加写锁,暂停读取.


##二、索引设计
```go
map[TermSign] *InvList
```
非常简单粗暴,使用golang自带的map类型存储倒排拉链.

##三、写入索引
```go
func (*MemoryIndex) WriteIndex(t TermSign,l *InvList) (error)
```
写入索引,内部加写锁进行写入.
同一个term多次写入,会进行append操作.


##四、读取索引
```go
func (*MemoryIndex) ReadIndex(t TermSign)(*InvList,error)
```
读取索引时先加读锁,然后读取拉链返回.




