---
layout: post
title: Binary Indexed Tree
date: 2025-11-11 02:36 +0000
categories: [Algorithm, Tree]
tag: [Binary Indexed Tree, HackerRank Array Manipulation]
math: true
---

### **What is Binary Indexed Tree**

The Binary Indexed Tree is used to solve the question of range sum query and point update. Giving a series of numbers, we suppose that we need the sum of index L to R, then we need to traverse the number of index L to R and sum them up. The time complexity is $O(n)$ or $O(R - L + 1)$, too slow. If we want to update a number of it, we just select it and change it, the time complexity is $O(1)$. So using the binary indexed tree to speed the range sum query is necessary.

### **The prerequisite knowledge**

- **Two's complement:** In computer, we use Tow's complement as the representation of negative number. And to get the two's complement, we start with the binary representation of its positive counterpart, flip the bits and add one.

- **lowbit Function:** All positive numbers' binary representation are like: $$0...01...10...01...10...0$$ if we flip the bits: $$1...10...01...10...01...1$$ and add one to get the negative one: $$1...10...01...10...10...0$$ and we perform the a bitwise AND operation between the positive one and negative one:
$$0...01...10...01...10...0$$ $$\&$$ $$1...10...01...10...10...0$$ We obtained the decimal value represented by the lowest set bit! The lowbit function just perform the whole process.

- **x - lowbit(x):** It won't stop removing the the lowest set bit step by step until the number equal to 0, which promises that that all numbers in $[1, n]$ can return to 0.

### **The Main Logic of BIT**

We need an array to contain a binary tree. Using lowbit function, the array will stores the partial sum for the specific range $[i - \text{lowbit}(i) + 1, i]$, where $i$ is the array index. The **x - lowbit(x)** promises that it will link to 0(ex. 7 -> 6 -> 4 -> 0). If we change $i$ to $i + k$, then we need to maintain the array, using $\text{lowbit} + x$ to find the partial sums which are influenced.  

### **Implement in c++**

```c++
int lowbit(x){
    return x & (-x);
}
void update(int x, int k)
{
    while(x <= n) // n is the ceil
    {
        tree[x] += k;
        x += lowbit(x);
    }
}
int query(int x)
{
    int res = 0;
    while(x > 0)
    {
        res += tree[x];
        x -= lowbit(x);
    }
    return res;
}
```

### **BIT with Difference Array**

The simple BIT can solve the question of range sum query and point update. How about the question of range updates and point queries? To achieve it, we need to import the Difference Array. The Difference Array contains like $a[i] - a[i - 1]$. More precisely, the difference array is used to store the difference between two elements. In the BIT, we primarily utilize the array storing the difference between adjacent elements to implement efficient range updates and point queries on the original array.

The tree array will contain $a[i] - a[i - 1]$. When we call query(i) function, the $\text{res}$ is like: $(a[i] - a[i - 1]) + (a[i - 1] - a[i - 2]) + ... + (a[1] - a[0]) = a[i] - a[0] = a[i]$ (Assuming a[0] is still 0, the array start from index 1). Then we achieve point query question. And observing the $\text{res}$, we can find that we can only update(add k or minus k) one number which in $[1, i]$, then the res will be update. In this case, we just call update(x, k) function, all the tree nodes that will return to x are updated, which promises the property of point query.

```c++
int lowbit(x){
    return x & (-x);
}
void update(int x, int k)
{
    while(x <= n) // n is the ceil
    {
        tree[x] += k;
        x += lowbit(x);
    }
}
int query(int x)
{
    int res = 0;
    while(x > 0)
    {
        res += tree[x];
        x -= lowbit(x);
    }
    return res;
}
int main(void){
    ...
    for(int i = 1; i <= n; ++i) 
    {
        cin >> a[i];
        update(i, a[i] - a[i - 1]); // Difference array
    }
    ...
    update(x, k); // range [x, y]
    update(y + 1, k); // removing the influence of the number > y.
    ...
}
```

### **Two BIT**

Going one step further, what if we want to implement range updates and range queries? We need to use BIT twice. Before it, we will do some mathematical deduction.

From the Difference array, we implement point query. To implement the range query, such as $[i - 2, i]$, we can use $\text{query}[i] + \text{query}[i - 1] + \text{query}[i - 2]$. In query(i), it is like $(a[i] - a[i - 1]) + (a[i - 1] - a[i - 2]) + ... + (a[1] - a[0]) = a[i] - a[0] = a[i]$. So we can get a formula: $$\sum_{i=1}^{x} \text{query}(i) = \sum_{i=1}^{x} \left( \sum_{j=1}^{i} d[j] \right) \\ = \sum_{j=1}^{x} d[j] (x - j + 1) = x \sum_{j=1}^{x} d[j] - \sum_{j=1}^{x} (j - 1) d[j]$$. This formula can get the sum of index $[1, x]$ (prefix sum).

So we need to maintain two BIT: query(x) for BIT_1 should product x; query(x) for BIT_2 should update in different way, for example: in range $[i, r]$ add $k$ should be add $k * (l - 1)$ and $-k * r$ to the number bigger than r to remove the influence, based on the formula.

```c++
vector<int>B1, B2;
long long lowbit(long long x)
{
    return x & (-x);
}
void update(vector<long long>& B, long long x, long long k)
{
    while(X <= n)
    {
        B[x] += k;
        x += lowbit(x);
    }
}
long long query(vector<long long>, long long x)
{
    long long res = 0;
    while(x > 0){
        res += B[x];
        x -= lowbit(x);
    }
    return res;
}
long long rangeUpdate(int l, int r, long long x)
{
    update(B1, l, k);
    update(B1, r + 1, -k);
    update(B2, l, k * (l - 1));
    update(B2, r + 1, -k * r);
}
long long preSum(long long x)
{
    return x * query(B1, x) - query(B2, x);
}
long long rangeSum(int l, int r)
{
    return preSum(r) - preSum(l);
}
int main(void)
{
    ...
    vector<long long> a(n + 1, 0);
    for(int i = 1; i <= n; ++i)
    {
        cin >> a[i];
        rangeUpdate(i, i, a[i]);
    }
    ...
    rangeUpdate(x, y, k);
    ...
    cout << rangeSum(x, y) << endl;
}
```

### **Time Complexity**

We only performed climbing the tree (updating) and descending the tree (summing/querying). Therefore, both the update and query operations in the BIT have a time complexity of $O(\log n)$.
