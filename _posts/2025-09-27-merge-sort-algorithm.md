---
layout: post
title: Merge Sort Algorithm
date: 2025-09-27 06:36 +0000
categories: [Algorithm, Sort]
tags: [Merge_Sort, Divide and Conquer]
math: true
---
### **Step1: What is merge?**

To understand the merge sort better, we need to learn merge first. Imagining two arrays which are sorted, we want to merge them to get a new sorted array. The simplest way is that compare with the same index, and the smaller enter the new array first, the bigger enter next. If the length of two arrays are not equal, when finishing comparing, we add the longer part to the new array directly. This method can be extended to the situation which have more sorted array, using recursion. **(Specially: If an array only have one number, then it is sorted. This theorem is very important!)**

### **Step2: Where is the merge sort from?**

If there is an array, such as [9, 4, 5, 1, 2, 6, 7], we can consider each number of the array as a sorted array. 9 is an array, 4 is an array, and so on. Then we set a new array to contain numbers waiting for merging. Finally feel free to merge the 'array'!!

### **Step3: Implement in c++**

**First**, focusing on the merge method.

``` c++
void merge(int start, int mid, int end, vector<int>& arr, vector<int>& temp)
    {
        int i = start; int j = mid + 1; int k = 0; // start to mid is left, mid to end is right
        while(i < mid + 1 && j < end + 1) // j = end + 1, because the size of an array minus one is the end of the array.
        {
            if(arr[i] < arr[j]) temp[k++] = arr[i++];
            else temp[k++] = arr[j++];
        }
        while(i < mid + 1) temp[k++] = arr[i++];// deal with the longer
        while(j < end + 1) temp[k++] = arr[j++];// deal with the longer
        for(i = start; i < end + 1; ++i) arr[i] = temp[i - start];
    }
```

**Second** To merge a number of arrays, it's significant to do partition. That means we should divided the array into two parts and do recursion.

```c++
void merge_sort(int start, int end,vector<int>& arr, vector<int>& temp)
    {
        if(start < end)
        {
            int mid = start + (end - start) / 2;
            merge_sort(start, mid, arr, temp);
            merge_sort(mid + 1, end, arr, temp);
            temp.resize(end - start + 1); // ensure temp is big enough.
            merge(start, mid, end, arr, temp);
        }
    } 
```

**Third**, the whole code:

```c++
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;
class MyVector{
public:
    vector<int> arr;
    MyVector(initializer_list<int> init) : arr(init){}
    void merge_sort(){
        if(arr.empty()) return;
        vector<int> temp(arr.size());
        ms(0, arr.size() - 1, temp);
    }
    
    void push(int value){
        arr.push_back(value);
    }
private:   
    void merge(int start, int mid, int end, vector<int>& temp)
    {
        int i = start; int j = mid + 1; int k = 0; // start to mid is left, mid to end is right
        while(i < mid + 1 && j < end + 1) // j = end + 1, because the size of an array minus one is the end of the array.
        {
            if(arr[i] < arr[j]) temp[k++] = arr[i++];
            else temp[k++] = arr[j++];
        }
        while(i < mid + 1) temp[k++] = arr[i++];
        while(j < end + 1) temp[k++] = arr[j++];
        for(i = start; i < end + 1; ++i) arr[i] = temp[i - start];
    }
    void ms(int start, int end, vector<int>& temp)
    {
        if(start < end)
        {
            int mid = start + (end - start) / 2;
            ms(start, mid, temp);
            ms(mid + 1, end, temp);
            merge(start, mid, end, temp);
        }
    } 
};

```

### How about the time complexity?

Because the recursion will split the array in half, the recursion depth is $\log_2 n$ -> $O(\log n)$. The merge operation will do $\mathcal{O}(n)$ operations, no matter the best, the worst or average. So the time complexity is $O(n \log n)$.
