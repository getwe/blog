title: goose自己写的搜索引擎-Value管理器
date: 2014-08-29 22:03:58
categories: 
- 技术
- 搜索引擎
tags:
- goose
- golang
- 搜索引擎
---

ValueManager在goose中负责管理`Value`数据.

<!-- more -->

##一、Value是什么
在goose的整个检索阶段,除了计算文本相关性,还可以利用额外的非文本信息对结果进行调权,影响排序.
这些跟doc相关的数据,就是Value.实际上就是一小段数据,在建库的时候随着doc一起建库,在检索中根据doc的唯一内部id获取.

如果了解过其它搜索引擎,总能找到类似的概念,比如[xapian](http://xapian.org/)中的[Values](http://getting-started-with-xapian.readthedocs.org/en/latest/concepts/indexing/values.html)

###实际例子
在一个检索系统,除了文本相关信息之外,希望把用户经常点击的结果往前排,那么可以将点击率写入Value,在检索的时候从Value解析得到点击率,进而对检索结果得分进行调权.

##二、设计思想
设计goose的Value机制的时候,我做了这样一些取舍:
###定长
在goose中,每个doc所关联的Value是定长的,具体长度在建库的时候根据外部配置文件决定.
定长Value在实现上比较方便,对我就是这样追求简约(我就是这么懒)
变长的Value可以根据每个doc的需要存储不同长度的数据,可以做到空间不浪费,但是随而带来的是系统的复杂度急剧提升,容易出错.在我实际的工程经验上,也很少有这方面的需求,定长Value在大多数情况下是足够的.

另外,Value的大小是由配置指定,意味着在整个索引库使用过程中不能随便改变其大小.建库的时候使用多大的Value,检索的时候也必须保持一致,不然程序可能崩溃.

###通用透明
为了保持一定的通用性,goose本身作为一个框架,完全不管Value里面写入的是什么数据,甚至于连具体什么格式都不管.
Value其实就是`[]byte`,一段二进制数据,建库的时候策略定制写入,检索阶段从其中解析数据.框架只负责管理读写.


##三、实现方法
Value的需求是需要在检索阶段能够快速读取,因此直接使用磁盘存储不太现实,完全放在内存中又可能会导致程序内存过大.因此在goose中简单采用mmap进行管理,兼顾一定的读写性能.

另外考虑到mmap文件可能太大,实际存储的时候会分成多个mmap文件,读写的时候根据id分到不同的文件去读写.

##四、读取Value
```go
// 读取value的引用,value只能进行读操作,任何写操作都是非法的
func (*ValueManager) ReadValue(inId InIdType)(Value,error)
```
读取接口根据内部id返回对应的Value.
由于go中没有类似c的`const char *`类型,目前这个接口有一个小缺陷,需要调用者自身保证不对Value进行写入操作.

##五、写入Value
```go
// 写入Value.可并发写
func (*ValueManager) WriteValue(inId InIdType,v Value)(error)
```
写入操作也比较简单,根据id确定写入文件以及文件偏移量然后写入Value.
