title: 基础算法目录
date: 2014-08-26 09:35:18
categories: 
- 技术
- 基础算法
tags:
- 算法
---

虽然工作这几年做产品做业务的比重很大,但是还是觉得那些最基础的教科书上的算法不能丢,平时有机会需要用到的情况下,会顺手把教科书上的算法简单实现,在需要用到的时候也经常是拿来简单一改即可使用.

这些代码小片段丢在磁盘上,万一磁盘坏了也算是一个杯具,这下可以借此机会存储到云端,存储有保障也可以在有需要时随时查阅,没有多少技术难度都是一些基础.

<!-- more -->

#查找算法
##[二分查找](/技术/基础算法/algorithm/binarySearch/)

#排序算法
##[经典排序:冒泡,选择,插入](/技术/基础算法/algorithm/classic-sort/)
##[快速排序](/技术/基础算法/algorithm/quickSort/)

#公共头文件
实现算法过程中,c++版本用到的一个方便调试的头文件
```cpp
#ifndef INTERFACE
#define INTERFACE
#include<iostream>
#include<math.h>
#include<algorithm>
#include<string.h>
#include<assert.h>

#include<stdio.h>
#include<stdlib.h>

#include<stack>
#include<vector>
#include<queue>
#include<map>

#include<bitset>
#include<iterator>


typedef std::vector<std::vector<int> > IntArrayArray;
typedef std::vector<std::vector<char> > CharArrayArray;

template <typename T>
void resize(std::vector<std::vector<T> > & c,
        int len1,int len2,T defvalue = T())
{
    c.resize(len1);
    for(int i = 0; i < len1;i++){
        c[i].resize(len2,defvalue);
    }
}
    
template<typename T>
void array_dump(std::vector<T> & cc)
{
    using namespace std;
    copy(cc.begin(),cc.end(),ostream_iterator<T>(std::cout," "));
    cout<<endl;
}

template<typename T>
void array_dump(T * begin,T * end)
{
    using namespace std;
    while (begin != end){
        cout<<*begin<<" ";
        ++begin;
    }
    cout<<endl;
}



template <typename T>
void ArrayArray_dump(std::vector<std::vector<T> > & c)
{
    using namespace std;
    for (int i=0; i<c.size();++i){
        for (int j = 0; j < c[i].size(); ++j){
            cout<<c[i][j]<<" ";
        }
        cout<<endl;
    }
}


template<typename T>
void myRand(T * array,unsigned int arrayLen)
{
    time_t t;
    time(&t);
    srandom(t);
    for(int i=0;i<arrayLen;i++){
        array[i] = random() % 1000;
    }
}

template<typename T>
void myRand(T ** pArray,unsigned int arrayLen)
{
    *pArray = new T[arrayLen];
    myRand(*pArray,arrayLen);
}

template<typename T>
void myRand(T ** pArray,char * strarrayLen)
{
    unsigned int len = atoi(strarrayLen);
    *pArray = new T[len];
    myRand(pArray,len);
}
#endif
```
