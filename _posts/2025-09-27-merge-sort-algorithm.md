---
layout: post
title: Merge Sort Algorithm
date: 2025-09-27 06:36 +0000
categories: [Algorithm, Sort]
tags: [Merge_Sort]
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
        while(i < mid + 1 && j < end + 1) // j = end + 1, because we consider the size of an array minus one as the end by default
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
//this adapt to n arrays more
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
void merge(int start, int mid, int end, vector<int>& index, vector<int>& temp)
{
    int i = start; int j = mid + 1; int k = 0;
    while(i < mid + 1 && j < end + 1)
    {
        if(index[i] > index[j]) temp[k++] = index[j++];
        else temp[k++] = index[i++];
    }
    while(i < mid + 1) temp[k++] = index[i++];/
    while(j < end + 1) temp[k++] = index[j++]; 
    for(i = start; i <= end; ++i) index[i] = temp[i - start];
}
void merge_sort(int start, int end, vector<int>& index, vector<int>& temp)
{
    if(start < end)
    {
        int mid = start + (end - start) / 2;
        merge_sort(start, mid, index, temp);
        merge_sort(mid + 1, end, index, temp);
        merge(start, mid, end, index, temp);
    }
}
int main(void)
{
    vector<int>arr;
    int times = 0; cin >> times;
    for(int i = 0; i < times; ++i)
    {
        int a = 0; cin >> a;
        arr.push_back(a);
    }
    vector<int>temp(arr.size());
    merge_sort(0, arr.size() - 1, arr, temp);
    for(int i = 0; i < arr.size(); ++i) cout << arr[i] << ' ';
    cout << endl;
}
```

### How about the time complexity?
- A: