title: goose自己写的搜索引擎-磁盘索引database.DiskIndex
date: 2014-08-23 14:51:31
categories: 
- 技术
- 搜索引擎
tags:
- goose
- golang
- 搜索引擎
---

DiskIndex模块用于管理磁盘倒排索引,是整个dabatase的最重要的一个类.

<!-- more -->

##一、设计思想
DiskIndex在实现的时候遵循了以下思想:

* 带状态模式,要不**只读**,要不**只写**.
* 读操作并发下安全,因此可以高效读取索引.
* 写操作不支持并发,内部通过加锁保证安全.
* 索引数据全部磁盘化,持久性存储由磁盘保证.

##二、带状态的磁盘索引
调用`NewDiskIndex()`构造函数生成一个\*DiskIndex,内部状态为`DiskIndexInit`.
如果构造函数后继续调用`Open`打开磁盘上已经存在的索引,则进入`DiskIndexReadOnly`索引只读状态,后续只能读取索引,不能写入.
如果构造函数后继续调用`Init`在磁盘指定位置创建全新的索引,则进入`DiskIndexWriteOnly`索引只写状态,后续只能写入索引,无法读取.
只读或者只写状态下调用`Close`后进入`DiskIndexClose`,此时所有资源已经清空释放,不能进行任何操作.


##三、索引设计
磁盘索引采用三层索引.

###三级索引  
三级索引是变长数据,按块写入,每一块数据就是完整的一条倒排拉链的数据.倒排拉链数据通过序列化后形成一整块二进制数据,然后整块写入.序列化的协议采用的是标准库`encoding/gob`协议.因此三级索引在磁盘上的格式就是:
| 第一个二进制数据块 | 第二块二进制数据块 | ... ... | 第N块二进制数据块

每一块数据所需要的索引信息就是在文件中的起始信息Offset以及数据长度Length,因此简单情况下在写入的时候记下`Offset,Length`,就可以在读取的时候正确读到数据.

在实际存储,三级索引可能是一个非常大的文件,文件大小可能会受到操作系统的限制,因此我采用了使用多个物理文件来组成逻辑上一个大文件,因此开发了`goose/utils/bigfile.go`来支持这个需求.通过这个改进以后,对于DiskIndex来说,写入三级索引需要记录的偏移信息就是`FileNo,Offset,Length`.

###二级索引  
二级索引是定长数据,负责存储`FileNo,Offset,Length`信息.直接采用一个系统文件进行读写,在golang中就是使用`*os.File`进行操作.二级索引每次读取或者写入9个字节,其中FileNo占用1个字节,Offset和Length分别各占用4个字节.

###一级索引  
一级索引是定长数据,顺序存储所有Term信息.采用mmap进行存储.

###零级索引  
零级索引是定长数据,常驻于内存,在只读索引中用到,用于快速定位一级索引,减少磁盘(mmap还是读磁盘)读取.

![磁盘索引结构](/static/img/goose-diskindex.png)


##四、写入索引
为了安全,写索引全程加互斥锁,保证不能并发写入.

```go
func (*DiskIndex) WriteIndex(t TermSign,l *InvList) (error)
```
由于可写的磁盘索引只能进行写入操作,因此所有写入的索引都是以此在磁盘中顺序存储.
另外,为了支持以后的索引读取,DiskIndex***要求所有顺序写入的TermSign是升序有序***

###写入三级索引
```go
func (*DiskIndex) writeIndex3(t TermSign, l *InvList) (error)
```
首先,利用`encoding/gob`协议把倒排拉链`*InvList`序列化为二进制块.

将序列化好的二进制块写入逻辑大文件中.

```go
func (*BigFile) Append(buf []byte)(*BigFileIndex,error)
```
返回的`BigFileIndex`就`FileNo,Offset,Length`.

###写入二级索引
把BigFileIndex压缩成一个9个字节的[]byte,根据已经写入的term数量算出接下来需要写入二级索引的文件偏移量,然后把二级索引信息写入.

```go
func (f *File) WriteAt(b []byte, off int64) (n int, err error)
```

###写入一级索引
最后写入TermSign信息,根据已经写入的term数量算出当前term在一级索引文件中需要保存的位置,然后直接保存.

##五、读取索引
对效率的高要求,索引读取支持并发读取,内部不会有任何锁操作.

```go
func (*DiskIndex) ReadIndex(t TermSign)(*InvList,error)
```

###读取零级索引
零级索引只存在于内存,在打开已有的磁盘的索引的时候根据一级索引实时生成.把一级索引按照定长分隔成块,每一块取第一个term组成零级索引.

读取索引的时候首先读取内存上的零级索引,确定所查找的term在一级索引中属于哪一块.

因写入的时候假设外部都是有序写入倒排拉链,因此零级索引是有序的,查找的时候进行一次二分查找,效率上可以得到保证.

```go
func (*DiskIndex) readIndex0(t TermSign) (int)
```

###读取一级索引
零级索引确定了term在一级索引中属于哪一块,接下来再在块中进行一次二分查找,最终确定term在一级索引中的位置.

###读取二级索引
确定了term在一级索引中的位置,便可直接读取二级索引信息,得到`FileNo,Offset,Length`.


###读取三级索引
最后,根据`FileNo,Offset,Length`读取三级索引,得到一块二进制块数据,对数据根据`encoding/gob`协议进行反序列化,最终得到倒排拉链.
