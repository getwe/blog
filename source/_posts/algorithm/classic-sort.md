title: 经典排序
date: 2014-08-26 10:09:23
categories: 
- 技术
- 基础算法
tags:
- 算法
- 排序
- 冒泡排序
- 选择排序
- 插入排序
---

经典的几个排序算法:冒泡,选择,插入.

<!-- more -->

```cpp
#include "interface.h"

// 冒泡排序,时间复杂度在n(n-1)/2 -- O(n^2)之间
void bubbleSort(int * array,unsigned int len)
{
    // end表示end之后的元素都是有序的
    unsigned int end = len - 1;
    while (end > 0){
        // 最后一次置换的元素,last之后的元素也是有序的
        int last = 0;
        // 每次循环从数组起始位置开始
        for(int i=0;i<end;++i){
            if (array[i] > array[i+1]){
                std::swap(array[i],array[i+1]);
                last = i;
            }
        }
        end = last;
    }
}


// 选择排序,认为是O(n^2),实现简单,待排序数组够小用选择挺好
void selectSort(int * array,unsigned int len)
{
    for (int i=0;i<len-1;i++){
        int min=i;
        for (int j=i+1;j<len;j++){
            if (array[j]<array[min]){
                min=j;
            }
        }
        std::swap(array[i],array[min]);
    }
}

// 插入排序,实际用途有吗?
void insertSort(int * array,unsigned int len)
{
    // 将最小元素放在最前面做观察哨
    int min=0;
    for (int i=0;i<len;i++){
        if (array[i]<array[min]){
            min=i;
        }
    }
    std::swap(array[0],array[min]);

    // i之前的元素都是符合目标的有序数组了
    // 紧接着把i后面剩下的每一个元素进行插入操作
    for (int i=1;i<len;i++){
        // j从待插入元素开始出发
        int j=i;
        // 暂存待插入元素
        int toInsert = array[j];
        // 前面的元素比当前位置的还大,就进行后遗一步操作
        while (array[j-1] > toInsert ){
            array[j] = array[j-1];
            j--;
        }
        // 否则,当前就是插入位置
        array[j] = toInsert;
    }
}


int main()
{
    using namespace std;
    const int arrayLen = 20;
    int array[arrayLen];
    myRand(array,arrayLen);
    cout<<"                ";
    array_dump(array,array+arrayLen);
    insertSort(array,arrayLen);
    cout<<"insert   Sort : ";
    array_dump(array,array+arrayLen);
    selectSort(array,arrayLen);
    cout<<"select   Sort : ";
    array_dump(array,array+arrayLen);
    bubbleSort(array,arrayLen);
    cout<<"bubble   sort : ";
    array_dump(array,array+arrayLen);
    cout<<endl;
    return 0;
}

```
