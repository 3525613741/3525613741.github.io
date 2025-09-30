---
layout: post
title: Insertion sort algorithm
date: 2025-09-29 09:56 +0000
categories: [Algorithm, Sort]
tags: [Insertion_Sort]
---
### **What is Insertion?**

Insertion sort is a simple sort algorithm. The basic logic of is that from the head to the tail of the array, if the previous one is larger than the next one, swap them and loop the process, achieving the insertion effct.

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

**Worst:** Every element need to swap to the head(reverse array). According to the arithmetic sequence sum formula, it equals to $\mathcal{O}(n ^ 2)$.

**Best:** Ascending array, it equals to $\mathcal{O}(n)$

**Average:** Random and unsorted array. Assuming the ith element, it can swap 1, 2, 3, ... i. So the average of swapping times is $\frac{\sum_{k=1}^{i} k}{i}$, equals to $\frac{i+1}{2}$. So from head to tail, it is $\sum_{i=1}^{n} \frac{i+1}{2}$. According to the arithmetic sequence sum formula, it equals to $\mathcal{O}(n ^ 2)$.
