---
layout: post
title: Disjoint-Set Union
date: 2025-10-15 08:59 +0000
categories: [Algorthm, Graph]
tags: [Disjoint-Set Union]
math: true
---

### **What is Disjoint_Set Union(DSU)?**

The DSU is a data struct that keeps track of collection of disjoint set and provides two functions:

- **find:** find which set an element x belongs to and return its root or representative of the set;

- **Union:** merge the sets that contain a and b.

### **Inplement DSU**

```c++
struct DSU{
    vector<int> parent;
    DSU(int n) : parent(n){
        for(int i = 0; i < n; ++i) parent[i] = i;
    }
    int find(int x)
    {
        while(x != parent[x]) x = parent[x];
        return x;
    }
    void union(int a, int b)
    {
        int pa = find(a); int pb = find(b);
        if(pa != pb) parent[pb] = pa;
    }
};
```

In this code, we can find that two areas to optimization:

- When the "find" function works, the 'x' will climb a long path to find its root;

- In some cases, attaching the smaller tree under the big one won't lift the height of tree. But attaching the big one under the smaller one will lift the height of tree absolutely.

So we will use path compression and control the union process to solve these two quesions.

### **DSU with oprimization**

```c++
struct DSU{
    vector<int> parent, size;
    DSU(int n) : parent(n), size(n, 1){
        for(int i = 0; i < n; ++i) parent[i] = i;
    }
    int find(int x)
    {
        if(x != parent[x]) parent[x] = find(parent[x]);
        return parent[x];
    }
    /* no recursion:
    (1) like recursion:
    int root = x;
    while(root != parent[root]) root = parent[root];
    while(x != root)
    {
        int next = parent[x];
        parent[x] = root;
        x = next;    
    }
    return root;

    (2) halve the path length each time,
    while(x != parent[x])
    {
        parent[x] = parent[parent[x]];
        x = parent[x];
    }
    return x;
    */
    void union(int a, int b)
    {
        int pa = find(a); int pb = find(b);
        if(pa == pb) return;
        if(size[pa] < size[pb]) swap(pa, pb);
        parent[pb] = pa;
        size[pa] += size[pb];
    }
}
```

In this way of using `parent[x] = find(parent[x]);` and `parent[x] = parent[parent[x]];`, every `parent` can connect to their root directly. Shorten the height that 'x' climbs.

### **Time Complexity**

- **Simple:** If the height of the tree is n, the x will climb n times in each operation. Suppose do m operations: $O(mn)$;

- **Path Compression:** Because every node connects to there root directly, so the time of find is close to 1. In here we use inverse Ackermann function $⍺(n)$ which is a very slowly growing function (for all practical purposes, ≤ 4), so the find runs in amortized $O(⍺(n))$ Suppose do m operations: $O(m\alpha(n)) \approx O(m)$.

- **Path Halving:** Less aggressive than full compression but still close to $⍺(n)$, the time complexity is also $O(m\alpha(n)) \approx O(m)$
