---
layout: post
title: Matrix Multiplication approach for All-Pairs Shortest Paths
date: 2025-10-18 06:18 +0000
catogries: [Algorithm, Graph]
tag: [APSP, Matrix]
math: true
---
### **How does it work?**

From the Floyd algorithm, we use $dist[i][j] = dist[i][k] + dist[k][j]$. In matrix, supposing the matrix A, we discuss the $A[i][j]$ and $A^2[i][j]$. According to the rule of matrix multiplication, we know that $A^2[i][j] = \sum_k (A[i][k] \cdot A[k][j]), k ϵ (1,dimension)$. So if we change the multiplication to plus and change the sum to minimize, we can simulate the Floyd-Warshall Algorithm.

Now defining a new multiplication named min-plus multiplication, the $A^2[i][j] = (A⊗A) [i][j] = min (A[i][k] + A[k][j])$. The meaning of $A^k[i][j]$ is there are k roads from i to j at most. When $k = n - 1$, we can finally solve the APSP.

### **Implement in c++**

- **Set a Matrix struct**

- **Define the min-plus multiplication**

- **Using Matrix exponentiation**

```c++
struct Matrix{
    ll n; vector<vector<ll>> matrix;
    Matrix(ll dimension) : n(dimension), matrix(dimension, vector<int>(dimension, INT_MAX)){}
};

Matrix multiply(const Matrix& a, const Matrix& b)
{
    int dimension = a.n;
    Matrix res(dimension);
    for(int row = 0; row < dimension; ++row)
    {
        for(int column = 0; column < dimension; ++column)
        {
            for(int temp = 0; temp < dimension; ++ temp)
            {
                res.matrix[row][column] = min(res.matrix[row][column], a.matrix[row][temp] + b.matrix[temp][column]);
            }
        }
    }
    return res;
}

Matrix matrixPow(Matrix A, int k)
{
    int dimension = A.n;
    Matrix res(dimension);
    for(int i = 0; i < n; ++i) res[i][i] = 0;
    while(k)
    {
        if(k & 1) res = multiply(res, A);
        A = multiply(A, A);
        k >>= 1;
    }
}
```
