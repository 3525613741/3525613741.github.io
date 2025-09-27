---
layout: post
title: Quick Sort Explaination
date: 2025-09-26 07:10 +0000
tags: [Algorithm, Sort]
---
### **Step1: Where is the quick sort from?**

In a out-of-order list, if we fixed one of them and name it "pivot", such as the middle one or the first one, then we divided the list into two parts(left and right). We put the one bigger than privot in the left part, put the one smaller than pivot in the right part. Repeat the process, we can finally get the sorted list.

### **Step2: What can we achieve it with c++?**

- **First**, we need the "pivot". Generally speaking, we often define the first one or the middle one of the list as pivot. And we should divide the list into two parts. That means that we need two pointers, which point the last first one and the last one.

> **(Attention, there are some difference of setting pointers between defining the first one and the middle one of the list as pivot)**

For the **first one**:

```c++
int i = l; int j = h; 
 /* 
That because we should keep the l and h unchanged.
The 'l' point the first index of the element of the array. 
The 'h' point the index after the last element of the array.
*/
int pivot = arr[i];


For the **middle one**:
```

```c++
int i = l; int j = h;
int middle = i + (j - i) / 2; // Avoid Overflow
int pivot = arr[middle];
/*
The 'l' point the index before the first element of the array. 
The 'h' point the index after the last element of the array.
*/
```

- **Second**, moving pointers to implement partition and swap the numbers.

```c++
while(i < j)
{
 do{
     ++i;
    }while(arr[i] > privot);
 do{
     --j;
    }while(arr[j] < privot);
}
if(i < j)
{
    swap(arr[i], arr[j]);
}
```

- **Third**, repeat the two steps above. YES! We can use recursion!

```c++
if(l < h)
{
    int j = partition(arr, l, h);
    quick_sort(arr, l, j);
    quick_sort(arr, j+1, h);       
}
```

- Finally, the whole code:

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
class Quick_Sort{
public:
    int partition(vector<int>&arr, int l, int h)
    {
        int privot = arr[l];
        int i = l; int j = h;
        while(i < j)
        {
            do{
                ++i;
            }while(arr[i] > privot);
            do{
                --j;
            }while(arr[j] < privot);
        }
        if(i < j)
        {
            swap(arr[i], arr[j]);
        }
        return j;
    }
    void quick_sort(vector<int>&arr, int l, int h)
    {
        if(l < h)
        {
            int j = partition(arr, l, h);
            quick_sort(arr, l, j);
            quick_sort(arr, j+1, h);       
        }
    }
};
```

### **Some questions**

- **Q**: Why should we use "do while" loop instead of "while loop" so that the pointers can point the first one and the last one?
- **A**: We can also use "while loop" absolutely, but if  we select the first one of the pivot, we need initially move the 'l' pointer to the next index before the loop started.

- **Q**: Why is it quick?
- **A**: Analyzing the time complexity:
