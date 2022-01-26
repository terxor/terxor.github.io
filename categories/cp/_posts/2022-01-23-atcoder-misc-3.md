---
author: terxor
title: Review of misc. problems - 3
category: cp
katex: true
---

*Source of these problems is [atcoder](http://atcoder.jp)*

### Problem abc-234-F

Given a collection of $N$ lowercase letters $a$ to $z$ (possibly containing duplicates), find
the number of strings that can be formed by using any number of these ($1$ to $N$).

We have $1 \le N \le 5000$

**Solution**

Let the letters be numbered $1, \ldots M$ with $1$ being $a$, and $M = 26$ being $z$.

Define $f(k, l)$ as number of strings of length $l$ that can be formed using letters 
$1, \ldots, k$.

Let $x_k$ be the frequency of letter $k$.

Then:

$$
f(k, l) = \sum_{0 \le i \le \min(l, x_k)} \left[ f(k - 1, l - i) \cdot \binom{l}{i} \right]
$$

which can be explained as: If we include $i$ occurences of letter $k$, then from total string
length $l$, we can choose $i$ places for these. Rest of the places will be occupied by string
formed by letters $1,\ldots,k-1$ of length $l-i$ (similar to *stars and bars* problem).

The final answer $X$ will be given as:

$$
X = \sum_{1 \le i \le N} f(M, i)
$$

We now validate this solution by finding upper bound on number of operations.
- Let $p_i = x_1 + \ldots + x_i$ (total count including letters $1, \ldots, i$). Then $p_M = N$.
- The number of computations required for letter $k$ is $O(x_k \cdot p_k)$ because
  for each $f(k, l)$ at most $x_k$ iterations are there and max length is $p_k$.
  We will use upper bound $O(x_k \cdot p_M) = O(x_k \cdot N)$ to make calculations simpler.
  
- Total number of operations is $O(x_1 \cdot N) + \ldots + O(x_M \cdot N) = O(p_M \cdot N) = O(N^2)$

***

### Problem abc-233-F

You are given a tree with $N$ vertices numbered $1, \ldots, N$. 
Initially vertex $i$ holds value $P_i$
where $P_1, \ldots, P_N$ is a permutation of $1, \ldots N$.

In one operation you can swap values of vertices along an edge.

Find a sequence of operations such that vertex $i$ has value $i$ with number of operations
not exceeding $\frac{N \cdot (N - 1)}{2}$.

> The original problem gives a permutation and pair of indices that can be swapped and asks for
  a sequence of operations to sort the permutation. It is reducible to the problem stated above.

**Solution**

It is always possible to reach the target state because there exists a path between each pair
of vertices in a tree.

We now show a way to achieve this within given constraints.

Let $V_i$ be the value on vertex $i$.
Let $M$ be the number of vertices ($M = N$ initially).

1. Find a leaf $x$.
1. If $V_x = x$ go to step 4.
1. Find path to vertex $y$ such that $V_y = x$. Swap values along this path starting
  from $y$ towards $x$. This will require at most $M$ operations (max path length in a tree).
1. Remove leaf $x$ and $M \leftarrow M - 1$
1. If $M > 0$ go to step 1.

The total number of operations, at most, is 
$N + (N-1) + \ldots + 1 = \frac{N \cdot (N - 1)}{2}$.

***