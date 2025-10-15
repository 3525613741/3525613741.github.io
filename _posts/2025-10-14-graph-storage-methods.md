---
layout: post
title: Graph Storage Methods
date: 2025-10-14 06:46 +0000
categories: [Algorithm, Gragh]
tag: [Graph storage]
---
### **Adjacency List**

There are two ways to achieve the Adjacency List: use vector or use linked list. Because the vector is automatic RAII container, it don't need to release the memory. But using linked list must release.

- **Using vector**

The vector simulates the behavior of a linked list throughout the process.

```c++
int main(void)
{
    int dots, edges; cin >> dots >> edges;
    vector<vector<pair<int, int>>>adjList(dots + 1);
    for(int i = 0; i < edges; ++i)
    {
        int num1, num2, weight;
        cin >> num1 >> num2 >> weight;
        adj[num1].push_back({num2, weight});
    }
}
```

- **Using linked list**

(1) release by hand:

```c++
struct Node{
    int to;
    Node* next;
    int weight;
    Node(int v, Node* n, int w) : to(v), next(n), weight(w){};
};
const int N = 100;
Node* head[N] = {nullptr};
void addEdge(int u, int v, int w)
{
    head[u] = new Node(v, head[u], w);// pointer head[u] type of Node point the address of the address of the new Node. If the same head[u] appears again, 
}
void releaseMemory(void)
{
    int len = sizeof(head) / sizeof(head[0]);
    for(int i = 0; i < len; ++i)
    {
        Node* p = head[i];
        while(p)
        {
            Node* temp = p;
            p = p -> next;
            delete temp;
        }
    }
}
int main(void)
{
    int dots, edges; cin >> dots >> edges;
    for(int i = 0; i < edges; ++i)
    {
        int u, v, w; cin >> u >> v >> w;
        addEdge(u, v, w);
    }
    releaseMemory();
}
```

(2) release by smart pointer:

```c++
struct Node{
    int to;
    shared_ptr<Node> next;// smart pointer, which can release the memory automatically.
    int w;
    Node(int v, shared_ptr<Node> n, int w) : to(v), next(n), weight(w){};
};
const int N = 100;
vector<shared_ptr<Node>> head(N, nullptr);
void addEdge(int u, int v, int w)
{
    head[u] = make_shared<Node>(v, head[u], w);// the "new" will turn head into a traditional pointer.
}
int main(void)
{
    int dots, edges; cin >> dots >> edges;
    for(int i = 0; i < edges; ++i)
    {
        int u, v, w; cin >> u >> v >> w;
        addEdge(u, v, w);
    }
}
```

**Adjacency List of Schematic Diagram**
![alt text](/assets/images/Adjacency_linked_list.jpeg)

### **Adjacency Matrix**

Using a n ✖️ n matrix to store the information.

```c++
int main(void)
{
    int dots, edges; cin >> dots >> edges;
    vector<vector<int>> matrix(dots + 1, vector<int>(dots + 1, INT_MAX));
    for(int i = 0; i < edges; ++i)
    {
        int u, v, w; cin >> u >> v >> w;
        matrix[u][v] = w;
    }
}
```

### **Chain Forward Star**

An efficient way to store graph.

```c++
struct Edge{
    int to, w, next;
}edge[1000];
int head[1000]; // consider it as a pointer.
int cnt; // stored the number of edges and used as the index to store the information of new edge;
void init(void)
{
    cnt = 0;
    for(int i = 0; i < 1000; ++i)
        head[i] = -1; // consider -1 as the nullptr by default.
}
void addEdge(int u, int v, int w)
{
    edge[cnt].to = v;
    edge[cnt].w = w;
    edge[cnt].next = head[u];
    head[u] = cnt++;
}// if don't use struct:
/*
to[cnt] = v;
w[cnt] = w;
next[cnt] = head[u];
head[u] = cnt++;
*/
int main(void)
{
    int dots, edges; cin >> dots >> edges;
    for(int i = 0; i < edges; ++i)
    {
        int u, v, w; cin >> u >> v >> w;
        addEdge(u, v, w);
    }
    // How to use it?
    /*
    for(int i = 1; i <= dots; ++i)
    {
        for(int j = head[i]; j != -1; j = edge[j].next)
    }*/
}
```

### **Graph Traversal**

For graph traversal, the Adjacency Matrix follows the rules of a 2D matrix, probing cell by cell; while Adjacency Lists (including the Array-Based Adjacency List, often called Forward Star or Chain-Forward Star) traverse all outgoing edges of one vertex before moving to the next.

- For **sparse graphs**, use an adjacency list or a Chain Forward Star.
- For **dense graphs**, use an adjacency matrix.
