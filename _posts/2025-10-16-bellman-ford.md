---
layout: post
title: Bellman Ford
date: 2025-10-16 12:24 +0000
categories: [Algorithm, Graph]
tag: [Single Source Shortest Path, Bellman Ford]
math: true
---

### **What is Bellman Ford**

The Bellman Ford algorithm is the algorithm that solving the single source shortest path questions. It also solve the problem that the Dijkstra can not deal with the graph with negative edge weights.

### **How does it work?**

Learning about the 'relax' in the Dijkstra. In the Bellman-Ford, it will relax all edges n - 1 times(n = dots). So it can not solve the Negative Weight Cycles because the distance will decrease continuous. But this feature give the function checking whether the graph have the negative weight cycles or not.

```c++
vector<vector<pair<int, int>>>adj;
void init()
{
    adj = vector<vector<pair<int, int>>>(1000);
}
void addEdge(int u, int v, int w)
{
    adj[u].push_back({v, w});
}
bool Bellman_Ford(int start, int dots)
{
    vector<int> dist(1000, INT_MAX);
    dist[start] = 0;
    for(int i = 0; i < dots - 1; ++i)
    {
        for(int u = 0; u < dots; ++u)
        {
            if(dist[u] == INT_MAX) continue;
            for(auto[v, w] : adj[u])
            {
                if(dist[v] > dist[u] + w) dist[v] = dist[u] + w;
            }
        }
    }
    for(int u = 0; u < dots; ++u)
    {
        if(dist[u] == INT_MAX) continue;
        for(auto[v, w] : adj[u])
        {
            if(dist[v] > dist[u] + w) return false;
        }
    }
    return true;
}
```

### **Time Complexity**
