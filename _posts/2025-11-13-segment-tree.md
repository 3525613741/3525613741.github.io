---
layout: post
title: Segment Tree
date: 2025-11-13 05:52 +0000
categories: [Algorithm, Tree]
tag: [Segment Tree]
math: true
---

### **What is Segment Tree?**

Segment Tree is a type of binary tree, which is used to solve the question in a range, such as finding the max one or min one in range L to R. The segment tree will divide the node [L, R] into left node [L, M] and right node[M + 1, R], M = (L + R) / 2 until L = R. So it's obvious that the tree node is range instead of the number. In this case, the container which contain this tree will be bigger. For example, if we have 4 numbers randomly, the L = 1, R = 4(Considering the index of array start from 1), so the root is [1, 4], then [1, 2], [3, 4] finally [1, 1], [2, 2], [3, 3], [4, 4]. The tree is like this:

![alt text](/assets/images/Segment_Tree.png)

So the size of container will equal to 2n - 1 + 1(index start from 1) = 2n. But to prevent the node are divided into more nodes, the space will be larger.

OK, from the image, we can see the segment tree's node is range, and the exact number can be the max, the min, the sum in the range!

Here is the method to build segment tree:

```c++
void build(int node, int l, int r, const vector<ll>& a){
    lazy[node] = 0;
    if(l == r)
    {
        maxTree[node] = a[l];
        minTree[node] = a[l];
        sumTree[node] = a[l];
        return;
    }
    int mid = (l + r) >> 1;
    build(node << 1, l, mid, a);
    build(node << 1|1, mid + 1, a); // index start from 1, so the son node is node * 2 and node * 2 + 1. 
    maxTree[node] = max(maxTree[node << 1], maxTree[node << 1 | 1]);
    minTree[node] = min(maxTree[node << 1], maxTree[node << 1 | 1]);
    sumTree[node] = maxTree[node << 1] + maxTree[node << 1 | 1];
}
```

### **Lazy Propagation**

It's weird of the `lazy[node] = 0` in build function. The lazy tree is the significant part of lazy Propagation. Lazy tree have same height and number of nodes but the exact number of node. Using lazy tree con record the operations in a range, such as add and assign. If adding 2 in range [L, R], the lazy tree will record 2 in all nodes which are excluded by [L, R]. If the [L, R] too small that will be included, only record on the node which excluded [L, R]. And when using the nodes that will be influenced by adding 2 before, the record will push down to confirm the property of the Segment Tree.

The c++ implement like this:

```c++
void pushdown(int node, int l, int r)
{
    if(lazy[node] != 0)
    {
        lazy[node << 1] += lazy[node];
        lazy[node << 1 | 1] += lazy[node];
        ll mid = (l + r) >> 1;
        sum[node << 1] += (mid - l + 1) * lazy[node];
        sum[node << 1 | 1] += (r - mid) * lazy[node];
        lazy[node] = 0;
    }
}
```

### **Operations in c++**

```c++
// range_add
void update(int node, int l, int r, int operateL, int operateR, ll k)
{
    if (operateL > r || operateR < l) return;
    if(operateL <= l && operateR >= r){
        lazy[node] += k;
        maxTree[node] += k;
        minTree[node] += k;
        sumTree[node] += (r - l + 1) * k;
        return;
    }
    pushdown(node, l, r);
    ll mid = (l + r) >> 1;
    if(operateL <= mid) update(node << 1, l, mid, operateL, operateR, k);
    if(operateR > mid) update(node << 1|1, mid + 1, r, operateL, operateR, k);
    sum[node] = sum[node << 1] + sum[node << 1|1];
}
ll queryMax(int node, int l, int r, int operateL, int operateR)
{
    if(operateL <= l && operateR >= r){
        return maxTree[node];
    }
    pushdown(node, l, r);
    int mid = (l + r) >> 1;
    return max(queryMax(node << 1, l, mid, operateL, operateR), queryMax(node << 1|1, mid + 1, r, operateL, operateR));
}
ll queryMin(int node, int l, int r, int operateL, int operateR)
{
    if(operateL <= l && operateR >= r){
        return minTree[node];
    }
    pushdown(node, l, r);
    int mid = (l + r) >> 1;
    return min(queryMin(node << 1, l, mid, operateL, operateR), queryMin(node << 1|1, mid + 1, r, operateL, operateR));
}
ll querySum(int node, int l, int r, int operateL, int operateR)
{
    if(operateL <= l && operateR >= r){
        return sumTree[node];
    }
    pushdown(node, l, r);
    ll mid = (l + r) >> 1;
    ll sumNum = 0;
    if(operateL <= mid) sumNum += query(node << 1, l, mid, operateL, operateR);
    if(operateR > mid) sumNum += query(node << 1|1, mid + 1, r, operateL, operateR);
    return sumNum;
}
```

### **Time complexity**

Building the tree, every node we will visit once and do $O(1)$ operations. So n nodes means $O(n)$. Updating the tree, if we only update one node, the time complexity is $O(logn)$ because we need to climb the tree. Querying in range, along the tree, we will visit $O(logn)$ layers, and visit two nodes in every layer at most. Updating in range, if we don't use lazy propagation, all nodes at most will be updated which do $O(1)$ operation which means $O(n)$. If we use lazy propagation, we only update $O(logn)$ times at most because we don't update the smaller range and just record the operation on lazy tree which means we only visit the left and right subtree in every recursion or just one node if the range of updating is small enough. The record operation and pushdown operation will be amortized in all operations.
