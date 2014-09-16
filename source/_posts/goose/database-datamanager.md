title: goose自己写的搜索引擎-Data管理器
date: 2014-09-09 15:57:28
categories: 
- 技术
- 搜索引擎
tags:
- goose
- golang
- 搜索引擎
---

DataManager管理在goose内部中的全部全文数据.

<!-- more -->

##一、Data是什么
在一个检索系统中,基本上都会有全文数据,用于存储每个Doc的全部信息,检索流程完成之后,将会返回每个Doc的全文数据,供其它系统使用.比如说前端利用这些全文数据渲染页面.
在我的实际工程中,也有使用到一些检索系统,不提供全文服务,在检索系统中只存储一个全局id,而在另外一个独立的分布式的key/value系统获取全文数据.设计goose的时候,直接支持了全文数据的获取,因为整个检索系统相对简单,不需要单独拉出来做一个模块.

命名的时候再一次参考了`Xapian`中的[Data](http://getting-started-with-xapian.readthedocs.org/en/latest/concepts/indexing/documents.html#document-data)概念,将全文数据命名为`Data`.

##二、设计思想

###变长数据
Data数据直接的特点就是变长,输入系统的每一个Doc肯定长度不一,存储的时候需要支持变长数据.

###通用透明
跟Value数据一样,为了保持通用性,goose中的Data就是一段二进制流`[]byte`,由策略决定如何使用.

###追加写
goose中对Data数据的写操作只支持在末尾追加写.
在建库以及增量更新阶段,goose将顺序写入Data数据.对于Data数据,只需要能够一直追加写入数据即可.由于Data是变长的,为了实现起来比较方便,不支持对Data文件进行修改和删除操作.如果对Data数据实在有修改需求,解决方法是再插入一个新的Doc,并在策略中根据外部ID做去重,这样做虽然造成了磁盘空间的浪费,但是机制比较简单.

###随机读
写操作限制比较多,读操作没有多少限制.功能上的需求要求能够随机读取数据,需要支持安全并发读取.

##二、实现方法

在实现上采用了跟[磁盘索引DiskIndex](/技术/搜索引擎/goose/database-diskindex/)非常类似的设计.如图采用了两层索引.

![Data磁盘数据结构](/static/img/goose-datamanager.png)

###二级索引
把每个写入的Doc的Data数据顺序写到磁盘文件,索引信息就是`FileNo,Offset,Length`.

###一级索引  
一级索引使用mmap实现,整个一级索引就是一个大数组,数组下标就是内部id,数组元素就是`FileNo,Offset,Length`.


##三、写入Data数据
```go
func (*DataManager) Append(inId InIdType,d Data) (error)
```
追加数据文件.不可并发写入,使用者应该自己做好并发控制.
同一个InId多次写入会进行覆盖操作,只有最后一次写操作数据有效,而且之前的写入的
数据会变成垃圾数据占用磁盘空间,无法删除.

把整块数据不经过任何处理,追加写入磁盘文件,再把磁盘索引信息写入一级索引.

##四、读取Data数据
```go
func (*DataManager) ReadData(inId InIdType,buf *Data) (error)
```
读取Data在检索阶段是一个高频动作,因此实现的读取是支持并发安全读取,内部不加锁.
在BigFile中,读取文件使用的是`os.File.ReadAt`方法,其底层是使用了`pread`读取,这个函数是并发安全的.

读取的时候需要外部传入一块空间足够的buf存放读取的结果.

##五、作业
给自己留一个作业,后续抽时间研究一下其它数据结构,支持磁盘数据的增删查改,在效率上有一定的折衷取舍.
