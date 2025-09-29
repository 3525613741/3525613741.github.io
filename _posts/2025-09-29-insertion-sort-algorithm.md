---
layout: post
title: Insertion sort algorithm
date: 2025-09-29 09:56 +0000
categories: [Algorithm, Sort]
tags: [Insertion_Sort]
---
### **What is Insertion?**

Insertion sort is a simple sort algorithm. The basic logic of is that from the begin to the end of the array, if the previous one is larger than the next one, swap them, achieving the insertion effct.

### **Implement by code**

```c++
void is(vector<int>& arr)
    {
        for(int i = 1; i < arr.size(); ++i)
        {
            int j = i;
            while(j > 0 && arr[j - 1] > arr[j])
            {
                swap(arr[j - 1], arr[j]);
                j--;
            }
        }   
    }
```

### **Time Complexity**
