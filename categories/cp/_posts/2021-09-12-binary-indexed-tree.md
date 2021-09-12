---
author: terxor
title: Binary Indexed Tree
category: cp
katex: true
---

**Binary Indexed Tree** (BIT) is a data structure which mainly
supports operations on ranges. It is also known as **Fenwick Tree**.

> BIT is similar to segment tree because:
> - Both have $O(\lg(N))$ height (for $N$ elements)
> - Both can be represented through arrays.
> - Both have nodes which represent a range of elements.
>
> However, there is a major difference:
> - Segment tree is a binary tree but not BIT (it is a *binomial tree*)

We illustrate the concept of BIT through the following example:

Let $A_1, \ldots, A_N$ be an array of integers.
There are two types of queries:
1. **Range query**: Given index $i$, find $i$-th prefix sum, i.e, $A_1 + \ldots + A_i$
2. **Point update**: Given index $i$ and value $s$ do: $A_i \leftarrow A_i + s$

Assume that there can be a large number of these queries.

**Solution**

We will use $F_0, \ldots, F_N$ to store sums of particular
ranges. We define $F$ now:
- $F_0 = 0$
- $F_i = A_{h(i) + 1} + A_{h(i) + 2} + \ldots + A_i$

where $h(x)$ is $x$ with its least significant bit removed.
$h(x)$ is defined for $x > 0$. It is obvious that $h(x) < x$
and thus $h(x) + 1 \le x$.

In the BIT there will be $N+1$ nodes $0,1,\ldots,N$.
- Node $i$ holds value $F(i)$.
- Node $0$ is the root.
- The parent of node $x$ ($x \neq 0$) is node $h(x)$.

For example, the parent of node
$6 = (110)_2$ is node $4 = (100)_2$. Parent of node $5 = (101)_2$ is also node $4$.
The parent of node $4 = (100)_2$ is node $0 = (000)_2$.

From this, one can observe that the length of the path from a node $x$
to the root is the number of bits in $x$. Since the maximum number of bits among
$0,1,\ldots,N$ can be at most $\lceil \lg(N) \rceil$, height of this tree
is $O(\lg(N))$.

The figure below shows a BIT for $N = 15$

![bit-tree-example](/assets/images/cp/bit-example.svg)

Implementation wise, simply an array is used to represent this tree.

For queries of type 1:
- Let $S = 0, j = i$
- Repeat until $j = 0$:
  - $S \leftarrow S + F(j)$
  - $j \leftarrow h(j)$

> To understand how this works, let $i = 13 = (1101)_2$
> - $F(13) = A_{13}$
>   - because $h(13) = 12$ (as $1101 \rightarrow 1100$)
> - $F(12) = A_{12} + A_{11} + A_{10} + A_{9}$
>   - because $h(12) = 8$ (as $1100 \rightarrow 1000$)
> - $F(8) = A_{8} + A_{7} + A_{6} + A_{5} + A_{4} + A_{3} + A_{2} + A_{1}$
>   - because $h(8) = 0$ (as $1000 \rightarrow 0000$)
>
> So, $F(13) + F(12) + F(8) = A_1 + \ldots + A_{13}$

To deal with updates (i.e., query type 2) we have to understand which nodes are affected
when we change the value at some index $K$, i.e, $A_K$, i.e, which nodes include $A_K$ in their
value? We have to add $s$ to each of those nodes to reflect this change.

We need to find all $x$ satisfying:

$h(x) < K \le x$ and $1 \le x \le N$

That is, node $x \ge K$ but on removing LSB from $x$ it becomes lesser than $K$

- For $x = K$ this holds.
- For $x > K$,
  lets look at the first bit from the *left* which differs in $x$ and $K$.
  It should be $1$ in $x$ and $0$ in $K$ (because $x > K$).

  ```
  x = [c_1,...,c_t]1[p_1,...,p_z]
  K = [c_1,...,c_t]0[q_1,...,q_z]
  ```

- Suppose LSB of $x$ is in the $p$ part, let it be some $p_i$. Then
  upon removing it will still hold that $x > K$. Thus $p_1 = \ldots = p_z = 0$.
  So we have:

  ```
  x = [c_1,...,c_t]1[0000000...0]
  K = [c_1,...,c_t]0[q_1,...,q_z]
  ```

  Note that there should be at least one $1$ in $q_1,\ldots,q_z$ to make
  $h(x) < K$.

which gives us:
- For some bit $i$ which is $0$ in $K$ (including leading zeros to make the total number
  of bits equal to those in $N$) *with at least one $1$ to right of it* (i.e. at some index less than $i$), we get an $x$ (call it $x_i$) by making:
  - all bits $j$ such that $j > i$ same as those in $K$
  - bit $i$ as $1$
  - all bits $j$ such that $j < i$ as $0$.

Suppose we obtained an $x_i$ (i.e., through bit $i$ which was $0$ in $K$).

Let bit $j$ be the first bit to the left of $i$ which is also $0$ in $K$.

There is a neat way of obtaining $x_j$ from $x_i$:
Observe that bits $i, i+1,\ldots,j-1$ will be $1$ in $x_i$ and bit $j$ as $0$.
Suppose we add $2^i$ to $x_i$, we get bit $j$ as $1$ and all bits after that $0$,
which is precisely $x_j$.


Let's take a look at an example to make it clearer without more technical treatment:
```
K   = 0101101101
    +          1
x_1 = 0101101110
    +         10
x_4 = 0101110000
    +      10000
x_7 = 0110000000
    +   10000000
x_9 = 1000000000
```

In simple words, switch to the next $0$ by adding LSB to $x$. Note that
$x$ should not exceed $N$.

Thus for query type 2, simply do:
- Let $x = K$
- Repeat until $x > N$:
  - $F_x \leftarrow F_x + s$
  - $x \leftarrow x + h(x)$

***

Let's look at another example appropriate for BIT

Let $A_1, \ldots, A_N$ be an array of integers.
There are two types of queries:
1. **Point query**: Given index $i$, find $A_i$
2. **Range update**: Given indices $l,r$ and value $s$ do: $A_i \leftarrow A_i + s$ $\forall$ $i=l,\ldots,r$

Assume that there can be a large number of these queries.

**Solution**

Let $B_1 \ldots B_N$ be some array such that $A_i = B_1 + \ldots + B_i$.

> We do not have to actually construct such an array, but the BIT will be based on $B$.

- The first query would be equivalent to the sum query in the first example.

- For second query, notice that on performing $B_k \leftarrow B_k + s$
  affects all of $A_k ...A_N$ (since $A$ is prefix sum array of $B$).
  Adding to a range $[l, r]$ is thus done in two steps (which are point updates from first example):
  - $B_l \leftarrow B_l + s$
  - $B_{r+1} \leftarrow B_{r+1} + s$ if $r+1 \le N$

To obtain $B$ initially:
- Initialize $B$ be to an array of $0$s (i.e., $A_i = 0$ $\forall$ $i=1,\ldots,N$)
- Then add $A_i$ to range $[i, i]$ (op. 2) $\forall$ $i=1,\ldots,N$.

***