---
layout: post
title: Master Theorem
date: 2025-10-12 06:24 +0000
categories: [Algorithm, Math]
tag: [master_theorem, Divide and Conquer]
math: true
---
### **What is Master Theorem?**

Master Theorem is an important tools to analyze the time complexity of divide-and-conquer algorithm, which form is: $$T(n) = aT\left(\frac{n}{b}\right) + f(n)$$

Symbol:

- $T(n)$: Total running time for input size n.
- $a$: Divide into 'a' subquestions.
- $b$: Size of each subquestions.
- $f(n)$: The cost of dividing time and merging result.(out of recursion)

### **How did the formula appear?**

In level 0, $T(n) = f(n)$, no recursion;
In level 1, $T(n) = a f\left(\frac{n}{b}\right) + f(n)$
In level 2, $T(n) = a^2f\left(\frac{n}{b^2}\right) + a f\left(\frac{n}{b}\right) + f(n)$
......
When should the recursion stop? Absolutely in the moment $\frac{n}{b^k} = 1$. After equivalent substitution, we get $ k = \log_b n $
According to the mathematical induction, in the level *k*,
$$T(n) = \sum_{k=0}^{\log_b n} a^k f\left(\frac{n}{b^k}\right)$$.

Through this summation formula, we can get the general form mentioned above $$T(n) = aT\left(\frac{n}{b}\right) + f(n)$$

### **How many subquestions does the question have?**

Suppose $f(n) = 1$, which means focusing on how many subquestions it divided. We get: $T(n) = \sum_{k=0}^{\log_b n} a^k$. According to the formula for summing a geometric sequence we get: $T(n) = \frac{a^{\log_b n + 1} - 1}{a - 1}$
When n -> +∞, $T(n) = {a^{\log_b n}}$
Base on the Change-of-Base Rule, knowing: ${a^{\log_b n}} = {n^{\log_b a}}$.
So there are ${n^{\log_b a}}$ subquestions.

### **How about the time complexity of $f(n)$?**

Initially we suppose $f(n) = \Theta\left(n^k \cdot (\log n)^p\right)$, which is a general formula that covers all situations.

There are ${n^{\log_b a}}$ subquestions, which means there are  ${n^{\log_b a}}$ operations at least. We just discuss the influence of $f(n)$.

- $Case 1:$
$f(n)$ is small enough: $k<{\log_b a}$
So there exist a **ε > 0**, that ensure $f(n) = O(n^{\log_b a - ε}{(\log n)}^p)$
According to the limit: $\lim_{n \to +\infty} \frac{(\log n)^p}{n^\varepsilon} = 0 \quad (p \in \mathbb{R}, \varepsilon > 0)$, knowing $n^ε ≫ (logn)^p$ => $n^{\log_b a - ε}{(logn)^p} < n^{\log_b a}$.
So the $f(n)$ have no influence. The time complexity is $O(n^{\log_b a})$. Then the time complexity of $T(n)$ is $Θ(n^{\log_b a})$

- $Case 2:$
$f(n)$ is balance: $ k = {\log_b a}$
In this case we need to discuss the value of p.
Based on $T(n) = \sum_{k=0}^{\log_b n} a^k f\left(\frac{n}{b^k}\right)$.
We can use integration to approximate this discrete summation:

$$
T(n) = \sum_{k=0}^{\log_b n} a^k \left( \frac{n}{b^k} \right)^{\log_b a} \log^p \!\left( \frac{n}{b^k} \right)
$$

$$
= \sum_{k=0}^{\log_b n} n^{\log_b a} \log^p \!\left( \frac{n}{b^k} \right)
$$

$$
= n^{\log_b a} \sum_{k=0}^{\log_b n} \log^p \!\left( \frac{n}{b^k} \right)
$$

Let $x = k$, then $x \in [0, \log_b n]$:

$$
T(n) = n^{\log_b a} \int_{0}^{\log_b n} \log^p \!\left( \frac{n}{b^x} \right) dx
$$

Let $t = \log \frac{n}{b^x}$, then $t \in [0, \log n]$, and $dt = -\log b \, dx$:

$$
T(n) = n^{\log_b a} \int_{\log n}^{0} t^p (-\log b)\, dt
$$

$$
= n^{\log_b a} \log b \int_{0}^{\log n} t^p \, dt
$$

If $p > -1$, then

$$
T(n) = n^{\log_b a} \log b \cdot \frac{(\log n)^{1+p}}{1+p}
= \Theta \!\left( n^{\log_b a} \log^{1+p} n \right)
$$

$f(1)$ is constant, so we don’t need to compute until $t = 0$.
We can stop at $c, c > 0$.

Else if $p = -1$:

$$
T(n) = n^{\log_b a} \log b \cdot (\ln \log n - \ln c)
= \Theta \!\left( n^{\log_b a} \log \log n \right)
$$

Else $p < -1$:

$$
T(n) = n^{\log_b a} \log b \cdot \frac{(\log^{1+p} n - c^{1+p})}{1+p}
$$

When $n \to +\infty,\, p < -1 \Rightarrow \log^{1+p} n - c^{1+p} \to \text{constant}$
$$
T(n) = \Theta \!\left( n^{\log_b a} \right)
$$

- $Case 3:$
$f(n)$ is large: $k > \log_b a$

$$
\Rightarrow f(n) = \Omega \!\left( n^{\log_b a + \varepsilon} \log^p n \right), \quad (\varepsilon > 0)
$$

$$
\text{If } p \le 0 \Rightarrow T(n) = \Theta \!\left( n^{\log_b a + \varepsilon} \right)
$$

$$
\text{Else } p > 0 \Rightarrow T(n) = \Theta \!\left( n^{\log_b a + \varepsilon} \log^p n \right)
$$
