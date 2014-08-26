title: 二分查找
date: 2014-08-26 09:51:59
categories: 
- 技术
- 基础算法
tags:
- 算法
- 二分查找
---

最简单的二分查找以及变形.

<!-- more -->

```cpp

#include "interface.h"

// 输入要求,数组前后闭空间[low,high],升序有序
// 二分搜索查找
int binarySearch(int array[],int v,int low,int high)
{
    while (low <= high){
        int mid = (low + high) / 2;
        if (v == array[mid]){
            return mid;
        }
        if (v > array[mid]){
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return -1;
}

// 输入要求,数组前后闭空间[low,high],升序有序
// 二分搜索查找,如果成功查找,返回坐标.如果没有查找到,返回比目标小,差距最小的元素
// 二分搜索的简单变形,有一些场景需要这样的要求
int binarySearchClosest(int array[],int v,int low,int high)
{
    int len = high;
    while (low <= high){
        int mid = (low + high) / 2;
        if (v == array[mid]){
            return mid;
        }
        if (v > array[mid]){
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    // while 最后一次开始的时候,low == high == mid
    // 如果v > array[mid] low = mid + 1
    // 变成 high == mid < low
    // 如果v < array[mid] high = mid - 1
    // 变成 high < mid == low
    // 结果都是array[high]是小于v且差距最小的一个
    assert(low == high + 1);
    int closest = high;
    return closest;
}

int main()
{
    using namespace std;
    const int arrayLen = 20;
    int array[arrayLen];
    myRand(array,arrayLen);
    std::sort(array,array+arrayLen);
    array_dump(array,array+arrayLen);
    int value = 0;
    do {
        cout<<"num to search : ";
        if (!(cin>>value)){
            break;
        }
        int pos = binarySearch(array,value,0,sizeof(array)/sizeof(int) - 1);
        cout<<"found pos : "<<pos<<endl;

        pos = binarySearchClosest(array,value,0,sizeof(array)/sizeof(int) - 1);
        cout<<"closest pos : "<<pos<<" closest num : "<<array[pos]<<endl;
    } while (1);
    return 0;
}

```
