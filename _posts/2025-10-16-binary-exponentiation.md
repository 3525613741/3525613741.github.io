---
layout: post
title: Binary Exponentiation
date: 2025-10-16 01:38 +0000
categories: [Algorithm, Optimization]
tag: [Binary Exponentiation, Bit Operation, Divide and Conquer]
---

### **What is Binary Exponentiation**

If we use `pow()` to compute $2^{13}$, the time complexity is not equal to $O(logn)$(look up the c++ document), and using simple loop is too slow. So the Binary Exponentiation is used to speed the exponentiation through the bit compute.

### **Some Prerequisites**

Suppose a integer 'a'.

- a & 1: if equal to 1, then a is odd; if equal to 0, then a is even;

- a >> k: right shift operator, if a = 1001, then a >> 1 = 0100.（Divide by two and floor the result）

- a << k: left shift operator, if a = 1001, then a << 1 = 10010. (double)

- Every decimal integer can be represented in binary. The binary bits are indexed starting from 0, reading from right to left. If a given binary bit equals 1, it represents $2^{\text{index}}$; if it equals 0, it represents 0. Summing these values restores the original decimal number.

### **The logic**

Focus on the exponent. If we express the exponent in binary, for example, $2^{13}$, $13 = (1101)_2$, so $2^{13} = 2^{1} \cdot 2^{4} \cdot 2^{8}$. From the prerequisites, we can use *a & 1* to check for parity, and use *a >> 1* to get every binary bit. Using both of them, we can check if the current binary bit is a 1 or a 0.

Focus on the base and set a container = 1. If the current binary bit is 1, then container ✖️ base to maintain this container and base ✖️ base to maintain the base; If the current binary bit is 0, just maintain the base.
Repeat this process through *a >> 1*.

Better understanding on image:

![alt text](/assets/images/Binary_Exponentiation.jpeg)

This method strongly resembles preprocessing, successfully reducing the original 13 operations to 6 operations (the seventh operation can be ignored since the result is already obtained by the sixth operation).

### **c++ code Implement**

Iterate version:

```c++
// #using ll = long long
double myPow(ll a, ll e)
{
    if(e == 0) return 1;
    if(e == 1) return a;
    ll res = 1; // the container
    bool negExp = false;
    if(e < 0)
    {
        negExp = true;
        e = -e;
    }
    while(e > 0)
    {
        if(e & 1) res *= a;
        a *= a;
        e >>= 1;
    }
    return negExp? 1.0 / res : res;
}
```

If we use recursion, the algorithm's logic is as follows: To compute $a^e$, we first compute $a^{e/2}$; to compute $a^{e/2}$, we first compute $a^{e/2/2}$, and so on, recursively, until the termination condition $e = 1$ is reached and a value is returned. At this point, we observe that if $e$ is an odd number, for example, $e = 9$, then $9/2 = 4$ (using integer division), and one base factor will be missing upon return. Therefore, we need to discuss odd and even numbers separately. When the exponent is even, the returned result's exponent is half of the original exponent, so the returned result must be squared. When the exponent is odd, we need to multiply the squared result by an additional base factor.
Recursion version:

```c++
// #using ll = long long;
double myPow(ll a, ll e)
{
    if(e == 0) return 1;
    if(e == 1) return a;
    bool negExp = false;
    if(e < 0)
    {
        negExp = true;
        e = -e;
    }
    int half = myPow(a, e / 2);
    if(e & 1) half = half * half * a;
    else half *= half;
    return negExp? 1.0 / half : half;
}
```

And this algorithm often be used in the Modular arithmetic:

```c++
//using ll = long long
//#define mod 1000000007
ll modPow(ll a, ll e)
{
    a %= mod;
    if(e == 0) return 1;
    if(e == 1) return a;
    ll half = modPow(a, e / 2);
    ll ans = (half % mod) * (half % mod) % mod;
    if(exp & 1) ans = (ans * a) % mod;
    return ans;
}
```

### **In Matrix**

In linear algebra, matrix exponentiation is frequently encountered. Therefore, as long as we can represent matrices and define the rules for matrix multiplication in code, we can utilize the Binary exponentiation algorithm to solve it.

```c++
// using ll = long long
const ll mod = 1e9 + 7;
struct Matrix{
    ll n; vector<vector<ll>> matrix;
    Matrix(int dimension, bool identity = false) : n(dimension), matrix(dimension, vector<ll>(dimension, 0)) {
        if(identity) 
        {
            for(int i = 0; i < n; ++i) matrix[i][i] = 1;
        }
    };
};

Matrix operator* (const Matrix& a, const Matrix& b)
{
    ll n = a.n;
    Matrix res(n);
    for(int row = 0; row < n; ++row)
    {
        for(int column = 0; column < n; ++column)
        {
            for(int temp = 0; temp < n; ++ temp)
            {
                if(a.matrix[row][temp] == 0 || b.matrix[temp][column] == 0) continue;
                res.matrix[row][column] = (res.matrix[row][column] + a.matrix[row][temp] * b.matrix[temp][column]) % mod;
            }
        }
    }
    return res;
}

Matrix matrixPow(Matrix base, ll exp)
{
    if(exp == 0) return Matrix(base.n, true);
    if(exp == 1) return base;
    Matrix half = matrixPow(base, exp / 2);
    half = half * half;// We did not reload the *=
    if(exp & 1) half = half * base;
    return half;
}
```

### **Time complexity**
