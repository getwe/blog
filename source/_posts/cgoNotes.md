title: cgoNotes
date: 2014-08-22 09:40:31
tags: 
- golang
- cgo
categories: 
- 技术
- golang
---

一些关于cgo的零散笔记.

<!-- more -->

#cgo编译
```go
/*
#cgo CFLAGS: -DDEBUG
#cgo LDFLAGS: -lpthead
#cgo pkg-config: libxml-2.0
#include <stdio.h>
struct persion{
};
*/
```
备注:

1. import "C" 前的注释相当于插入一个c头文件
2. \#cgo 相当于设置cgo的环境变量
3. `CGO_CFLAGS`和`CGO_LDFLAGS`加入编译链接选项,一般只定义一些通用的选项
4. CGO也支持pkg-config语法

#关键字冲突
在go文件中,C标识符或者成员变量如果在go中是关键字,会导致命名冲突.
可以通过添加前缀"_"获取,假设c struct如果有成员变量名字为type,在go使用该变量会导致冲突
必须这样使用xxx_struct_object._type

#c的字符串跟go的string转换
通过复制的方式实现的一些转换函数
```go
// Go string to C string
// C.free() 需要调用保证生成的c字符串释放内存
func C.CString(string) *C.char
 
// C string to Go string
func C.GoString(*C.char) string
  
// C string,leng to Go string
func C.GoStringN(*C.char,C.int) string
   
// C pointer,length to Go []byte
func C.GoBytes(unsafe.Pointer,C.int) []byte
```

#调用c函数传递一块内存
c函数需要的func(char * buf,int len)这样一块内存进行操作,go传递的方法
```go
buf := make([]byte,10240)
C.func( (*C.char)(unsafe.Pointer(&buf[0])) , C.int(len(buf)) )
```

#标准C数值类型在go中的类型
| *go*        | *c*                |
|-------------|--------------------|
| C.char      | char               |
| C.schar     | signed char        |
| C.uchar     | unsigned char      |
| C.short     | short              |
| C.ushort    | unsigned short     |
| C.int       | int                |
| C.uint      | unsigned int       |
| C.long      | long               |
| C.ulong     | unsigned long      |
| C.longlong  | long long          |
| C.ulonglong | unsigned long long |
| C.float     | float              |
| C.double    | double             |
| void*       | unsafe.Pointer     |

#cgo在命令行传递选项给c编译器
```bash
go tool cgo [compiler options] file.go
```
[compiler options]传递给gcc,作为编译c代码的编译选项




