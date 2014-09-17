title: goose自己写的搜索引擎-静态索引
date: 2014-09-17 21:01:37
categories: 
- 技术
- 搜索引擎
tags:
- goose
- golang
- 搜索引擎
---

静态索引StaticIndex在goose中更接近业务逻辑,表示的是由其它模块生成好的只读索引.

<!-- more -->

StaticIndex在检索阶段中使用,提供只读索引服务.在goose中静态索引就只是由磁盘索引组成.
StaticIndex所设计的功能就是打开磁盘已存在的索引然后提供只读索引操作.

所提供的读取方法也是直接使用DiskIndex的读取方法.
```go
// 读取索引
func (this *StaticIndex) ReadIndex(t TermSign)(*InvList,error) {
    return this.disk.ReadIndex(t)
}
```

