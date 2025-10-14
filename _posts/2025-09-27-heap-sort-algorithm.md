---
layout: post
title: Heap Sort Algorithm
date: 2025-09-27 11:49 +0000
categories: [Algorithm, Sort]
tags: [Heap_Sort]
math: true
---
### **Step1 What is heap?**

To understand the definition of the heap, it is necessary to learn about binary tree, especially the complete binary tree.
From Wiki:

- Binary Tree: In computer science, a binary tree is a tree data structure in which each node has at most two children, referred to as the left child and the right child.

- Complete Binary: A complete binary tree is a binary tree in which every level, except possibly the last, is completely filled, and all nodes in the last level are as far left as possible. It can have between 1 and 2h nodes at the last level h.

So the heap is a complete binary tree. At here, if you learn about binary tree, I will give you a simple way to memorize the complete binary tree: The last level dad have left son, have two sons, or have no son. Another level dad must have two sons. And No more than one father only has one left son.

There are two types of heap: Max Heap and Min Heap. The max heap means that the largest one must at the root of the tree and the value of parent node should greater than or equal to the values of its children. Min heap is the opposite of the max heap.

### **Step2 How to max heapify"**

As we know that, we need a max heap. So the question is: How can we get the max heap? This process is max heapify. Attention: the index of the tree node start from 0, the index of root is 0.

Creating an index complete binary tree:
![alt text](/assets/images/Binary_Tree.png)
In the image, we find that the index of every left son equals to 2 times of its dad's index and plus 1. And in max heap, the dad's value should be greater than the value of its son(s). Along this way, we can start to swap.

The specific operations are follows:

```c++
void max_heapify(int start, int end, vector<int>& arr)
    {
        int dad = start; int son = 2 * dad + 1;
        while(son < end)
        {
            if(son + 1 < end && arr[son] < arr[son + 1]) son++; // son + 1 means the right son. Finally give the greatest son.
            if(arr[dad] >= arr[son]) return;
            else{
                swap(arr[dad], arr[son]);
                dad = son;
                son = dad * 2 + 1;
            }
        }
    }
```

### **Step3 Implement Heap Sort**

Initailly, we should heapify from the last dad. From son = dad * 2 + 1, we know that dad = son / 2 - 1. Using loops, we can heapify from the bottom to the root.

After max heapfying, the largest number is at the root. At this time, we just swap the first number(root) and the last number. The largest one is at the end, and next we exclude it.

Repeat this process(use loops), we can get a sorted array.

```c++
void heap_sort(vector<int>& arr)
    {
        int len = arr.size();
        for(int i = len / 2 - 1; i > 0; --i) max_heapify(i, len, arr);
        for(int i = len - 1; i >= 0; --i)
        {
            swap(arr[0], arr[i]);
            max_heapify(0, i, arr);
        }
    }
```

### **The whole code**

```c++
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;
class Heap_Sort{
    public:
    void max_heapify(int start, int end, vector<int>& arr)
    {
        int dad = start; int son = 2 * dad + 1;
        while(son < end)
        {
            if(son + 1 < end && arr[son] < arr[son + 1]) son++; // son + 1 means the right son. Finally give the greatest son.
            if(arr[dad] >= arr[son]) return;
            else{
                swap(arr[dad], arr[son]);
                dad = son;
                son = dad * 2 + 1;
            }
        }
    }
    void heap_sort(vector<int>& arr)
    {
        int len = arr.size();
        for(int i = len / 2 - 1; i >= 0; --i) max_heapify(i, len, arr);// Initail maintain from the bottom, and obtain the maximum numbers, at the same time ensuring that the second largest number is on the second level, the third largest number is on the third level, and so on.
        for(int i = len - 1; i >= 0; --i)// After that, we continue to maintain it from top, ensuring that the parent value is greater than the child value at each level.
        {
            swap(arr[0], arr[i]);
            max_heapify(0, i, arr);
        }
    }
};
```

### **Time complexity**

The heap is the Binary Tree, whose height is $\log_2 n$ -> O($\log n$). In every level, we need to compare and swap $O(1)$ times. And we need to call max heapify n times to exclude the max element. So the time complexity is $O(n \log n)$, no matter the best, the worst, and the average.
