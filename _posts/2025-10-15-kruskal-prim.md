---
layout: post
title: Kruskal & Prim
date: 2025-10-15 04:56 +0000
categoried: [Algorithm, Graph]
tag: [MST, Kruskal, Prim]
---

### **What is MST?**

Given an undirected, connected, weighted graph, an MST is a subset of edges that:

- connect all vertices
- has no cycles
- and the minimum possible total edge weight

### **MST Algorithm**

There are three algorithms to solve the MST problems and all of them are based on the Greedy Algorithm:

- **Prim**

The logic of Prim is similar to the BFS, steps:
(1) Pick any starting vertex;
(2) Find the smallest edge that connects the vertex in the MST to a vertex outside it;
(3) Add this vertex and edge;
(4) Repeat until all vertices are included.

```c++
vector<vector<pair<int, int>>> adj;
int Prim(int n, int start)
{
    vector<int> parent(n, -1);
    vector<int> mst;
    vector<int> dist(n, INT_MAX);
    dist[0] = 0;
    vector<bool> visit(n, false);
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> pq;
    pq.push({0, start});
    int total = 0;
    int count = 0;
    while(!pq.empty() && count < n - 1) // when count = n - 2, if push into the mst, the number of edges will be n - 1, so it can stop.
    {
        auto [d, u] = pq.top(); pq.pop();
        if(visit[u]) continue;
        visit[u] = true;
        total += d;
        if(parent[u] != -1)
        {
            mst.push_back({parent[u], u});
            count++;
        }
        for(auto [v, w] : adj[u])
        {
            if(!visit[v] && dist[v] > w)
            {
                dist[v] = w;
                parent[v] = u;
                pq.push({dist[v], v});
            }
        }
    }
    return total;
}
```

*The explanation of this code:*

The graph is stored using an adjacency list. A priority queue based on a min-heap (using std::priority_queue<... , std::vector<...>, std::greater<...>>) is used to store the weight of each edge and its destination vertex. Edges with smaller weights take priority, and if weights are equal, the edge with the smaller vertex ID takes priority.

Similar to unlocking a fog-of-war area in a sandbox game where details are only revealed upon exploration, the priority queue only stores edges originating from "unlocked areas" (visited vertices). Therefore, the priority queue only contains elements from the "unlocked region," and the element popped first will inevitably be the one with the smallest weight within the "unlocked region."

Pop the top element of the priority queue, mark the destination vertex as visited, and then traverse its neighbors using the adjacency list, which is equivalent to "unlocking the area".

If a traversed edge satisfies two conditions—its weight is smaller than the previously stored weight to reach its destination vertex, and its destination vertex has not been visited—then this edge (weight and destination vertex) is pushed into the priority queue. Since the priority queue is sorted by weight in ascending order, when we select this edge to enter the Minimum Spanning Tree (MST), it means there cannot be any other edge with a smaller weight that can connect to this vertex, so no further pushing to the queue for this vertex is needed at that moment.

The iterative process of popping and pushing is essentially finding and connecting the edge with the minimum weight from the "unlocked area" each time. In this way, each new vertex is connected by the minimum-weight edge originating from the set of visited vertices. The use of the visited flag ensures that no cycles are formed.

- **Kruskal**

The logic of Kruskal is that select the edge having the smallest weight every time and unite nodes the edge connected. Being a circle means that the two nodes that united just now have the same root. So using Disjoint-set union not only can merge nodes but also can check whether they have the same root or not.

```c++
struct Edge{
    int u, v, w;
    Edge(int start, int end, int weight) : u(start), v(end), w(weight) {};
};

struct Cmp{
    bool operator()(const Edge& a, const Edge& b){
        return a.w > b.w;
    }
};

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
    bool union(int a, int b)
    {
        int pa = find(a); int pb = find(b);
    }
};

int Kruskal(int n, vector<Edge>& edges)
{
    priority_queue<Edge, vector<Edge>, Cmp>pq;
    for(auto& e : edges) pq.push(e);
    DSU(n);
    int count = 0; // count = n - 1, all the nodes are connected, then stop.  
    int total = 0;
    while(!pq.empty() && count < n - 1)
    {
        Edge tmp = pq.top(); pq.pop();
        if(union(tmp.u, tmp.v))
        {
            count++;
            total += tmp.w;
        }
    }
    return total;
}
```

In here, maybe you have some questions about the logic of `struct Cmp{}`. Checkint the cpp document to learn how the priority_queue inplemented, you will know.
