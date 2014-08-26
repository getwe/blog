title: 快速排序
p: algorithm/quicksort.md
date: 2014-08-26 15:05:29
categories: 
- 技术
- 基础算法
tags:
- 算法
- 排序
- 快速排序
---

快速排序的递归实现和利用栈数据结构实现.

<!-- more -->

```cpp
#include "interface.h"


// [low,high]
// 从小到大排序
int partition(int * array,unsigned int low,unsigned int high)
{
    using namespace std;
    // 取中间一个数作为支点,避免出现最坏情况
    std::swap(array[low],array[(low+high)/2]);

    unsigned int pivotkey = array[low];
    // 要点:分区算法的退出在于low == high 三个循环都需要
    while (low < high){
        // 找到比支点还小,换到前面半区
        while(low < high && pivotkey <= array[high])
            --high;
        array[low] = array[high];
        // 找到比支点还大,换到后面半区
        while(low < high && array[low] < pivotkey)
            ++low;
        array[high] = array[low];
    }
    // low == high
    array[low] = pivotkey;
    return low;
}
// [0,arrLen)
void quickSort(int * array,unsigned int arrayLen)
{
    if (arrayLen > 1){
        unsigned int mid = partition(array,0,arrayLen-1);
        quickSort(array,mid);
        quickSort(array+mid+1,arrayLen-mid-1);
    }
}

// [0,arrlen)
// 利用栈实现
void quickSort_stack(int * array,unsigned int arrayLen)
{
    using namespace std;
    // [first,second]
    typedef pair<unsigned int,unsigned int> LR; 
    stack<LR> SortStack;


    SortStack.push(make_pair(0,arrayLen-1));
    while (!SortStack.empty()){
        LR lr = SortStack.top();
        SortStack.pop();

        if (lr.second > lr.first){
            unsigned int mid = partition(array,lr.first,lr.second);
            SortStack.push(make_pair(lr.first,mid));
            SortStack.push(make_pair(mid+1,lr.second));
        }
    }
}

int main()
{
    using namespace std;
    const int arrayLen = 20;
    int array[arrayLen];
    int arr1[arrayLen];
    int arr2[arrayLen];
    int arr3[arrayLen];

    myRand(array,arrayLen);
    cout<<"                    : ";
    array_dump(array,array+arrayLen);

    for(int i=0;i<arrayLen;i++){
        arr1[i] = arr2[i] = arr3[i] = array[i];
    }

    cout<<"std::sort           : ";
    sort(arr1,arr1+arrayLen);
    array_dump(arr1,arr1+arrayLen);

    cout<<"quick sort (resurse): ";
    quickSort(arr2,arrayLen);
    array_dump(arr2,arr2+arrayLen);

    cout<<"quick sort (stack)  : ";
    quickSort_stack(arr3,arrayLen);
    array_dump(arr3,arr3+arrayLen);
    cout<<endl;
    return 0;
}
```





