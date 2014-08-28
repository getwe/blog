title: 第N小的数
date: 2014-08-28 22:02:14
categories: 
- 技术
- 面试算法
tags:
- 算法
- 快速排序
- 分治法
---
一个常见于面试中题目,**查找第N小的元素**.

<!-- more -->

印象在实际工作中还没使用过,就是在这些年的招聘中问过好几次这个问题,所以在这里分类给分到面试算法中.

#越简单越好
简单才好,效率不是最高和可以接受.如果数组真的不大,干嘛不进行一次全排序,然后输出结果呢?

```cpp
std::sort(array,array+arrayLen);
std::cout<<array[num]<<std::endl;
```
代码简单,用任何一种语言都会有对应的基础库,直接做个排序输出,这就是最佳答案!

#分治解决

如果说需要处理的数组非常大,以至于全排序需要浪费很多计算资源,做了很多没必要的排序.
那么改进的思路就是利用快排类似的做法,快速定位.
首先,需要一个跟快排完全一样的分区算法:
```cpp
// 跟快速排序所用到的partiton方法完全一样
// [low,high]
// 从小到大排序
int partition(int * array,unsigned int low,unsigned int high)
{
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
```
这样分区后,可以把数组分成两块,然后再看需要在的数字是在左半部分还是右半部分,然后再处理对应的部分,另外一部分就可以完全忽略了.
```cpp
// 快速局部排序,找到第N小的数,其它元素不一定有序
// N取值范围是[0,arrayLen)
bool findMinN(int * array,unsigned int arrayLen,int num)
{
    if (num < 0 || num >= arrayLen){
        return false;
    }
    int mid = -1;
    int left = 0;
    int right = arrayLen-1;
    while (mid != num){
        mid = partition(array,left,right);
        if (num < mid)
            right = mid - 1;
        if (num > mid)
            left = mid + 1;
    }
    return true;
}
```

#堆排序
以上的解法都要求全部数据都需要导到内存进行处理,如果数组特别大,那么就不可取了.
M个数求第N小的数.如果N个数内存完全放得下,那么可以建一个最大堆来处理.
依次读入M个数,保持维护一个最大堆,全部处理完成后,堆剩下的就是最小的N个数,而堆顶就是第N小个数.
```cpp
// 使用堆辅助实现
// 快排实现需要把全部数据在内存中进行排序,只是不需要做全排而已
// 如果全部数据在内存放不下,而N个数在内存中放得下,应该使用堆实现
// 演示方便,还是使用纯内存数组
// N取值范围是[0,arrayLen)
bool findMinNHeap(int * array,unsigned int arrayLen,int num)
{
    using namespace std;
    if (num < 0 || num >= arrayLen){
        return false;
    }
    
    // 先把前num+1个元素建成大顶堆
    make_heap(array,array+num+1);

    for (int i=num+1;i<arrayLen;i++){
        if (array[i] > array[0]){
            continue;
        }

        pop_heap(array,array+num+1);
        array[num] = array[i];
        push_heap(array,array+num+1);
    }

    // 堆里面保留了最小的n个元素
    // 堆顶就是第n小个数
    swap(array[0],array[num]);
    return true;
}
```

#标准库算法
如果是使用c++编程,标准库里面就有一个算法可以直接使用,如果是真正工程上的问题,理解解决问题的思路,然后用现成的吧
```cpp
nth_element(array,array+num,array+arrayLen);
cout<<array3[num]<<endl;
```


