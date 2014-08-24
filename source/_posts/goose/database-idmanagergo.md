title: goose自己写的搜索引擎-ID管理器
date: 2014-08-24 20:39:20
categories: 
- 技术
- 搜索引擎
tags:
- goose
- golang
- 搜索引擎
---

IdManager在整个索引库中负责管理内外部文档id.

<!-- more -->

##一、设计思想

一般的检索系统,为了方便内部管理,会为外部文档分配一个内部id,存在于检索内部,每一个doc都会有一个唯一id.
相对应的,外部输入的每一篇文档会对应有一个外部id,这依赖整个系统其它子系统对id的管理.

存在着内部id与外部id的互相映射关系,但是在goose的实现中,只实现了内部id查找外部id的功能,不支持在检索中使用id查找内部id,因为goose所需要实现的功能暂时不需要.

为了实现内部id快速查找外部id,需要可安全并发查询的key/value算法.另外Id关系数据要有比较高(相对于纯内存数据)的安全性,可支持并发读写.

##二、实现方法
直接使用磁盘文件实现key/value算法.内部id在goose中顺序分配,是一个int32类型.goose设计解决百万级别的doc数,要求外部id也是一个int32类型.
将内部id作为文件偏移量信息,对应的数据作为外部id.

假设最多有1千万个文档id,每个id使用4个字节存储,只需要1000\*10000\*4/1024/1024 = 38MB的文件便可以存储映射信息,另外由于文件体积较小,直接使用mmap方式打开,方便读写以及同步到磁盘.

##三、分配id
```go
func (this *IdManager) AllocID(outId OutIdType) (InIdType,error)
```
分配id不可并发,内部进行加锁以保证安全.

内部id顺序分配,根据下一个可分配的内部id乘于4得到文件偏移量,在文件中写入外部id.


##四、查找id
```go
func (this *IdManager) GetOutID(inId InIdType)(OutIdType,error)
```
每一个外部id占用4个字节,使用内部id乘于4得到文件偏移量,直接读取得到外部id.






