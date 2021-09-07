---
author: terxor
title: Review of misc. problems - 1
category: cp
katex: true
---

*Source of these problems is [atcoder](http://atcoder.jp)*

### Problem abc-174-E

We have $N$ logs of length $A_1, \ldots, A_N$.
We can cut these logs at most $K$ times in total.
When a log of length $L$ is cut at a point whose
distance from an end of the log is $t$ ($0 < t < L$)
, it becomes two logs of lengths $t$ and $L - t$.

Find the shortest possible length of the longest log
after at most $K$ cuts (rounding up to an integer)

We have $1 \le A_i, K \le 10^9$, $1 \le N \le 10^5$

**Solution**

A greedy solution involving assignment for each cut
may not run in time because of upper bound of $K$.

Let $f(x)$ be the minimum number of cuts required to 
obtain minimum length of longest log equal to 
$x$ (rounded up).

It is clear that $f(x) \ge f(y)$ $\forall$ $x < y$. In
other words, $f$ is a non-increasing function. We need
to find the minimum $x$ such that $f(x) \le K$. Thus a
solution with binary search is possible.

Computation of $f(x)$ is straightforward and
it requires $O(N)$ time: For each log,
find minimum number of cuts to get length of smallest
piece $x$ (we do not elaborate here why cutting the log
into parts of equal length leads to the minimum 
length of longest piece)

The overall time complexity is $O(N \log{A_{max}})$ 
because for binary search a comfortable upper 
bound for $x$ can be taken as $A_{max}$.

***

### Problem abc-164-D

Given is a string $S$ consisting of digits from
$1$ to $9$ and an integer $K$.

Find the number of pairs of integers $(i, j)$
$(1 \le i \le j \le |S|)$ that satisfy the following
condition:

In base ten, the $i$-th through the $j$-th
characters of $S$ form an integer that is a multiple
of $K$

We have: $1 \le |S| \le 200000$,
$1 \le K \le 10^9$, $K$ is coprime with 10

**Solution**

Let $g(i, j)$ be the number represented by substring
$S_{i \ldots j}$ ($i \le j$)

We will define $g$ in terms of another function $f$

Let $f(i)$ be the number represented by prefix $i$
of string $S$ (i.e, $S_{1 \ldots i})$.

We have:

$$
g(i,j) = f(j) - f(i-1) \cdot 10^{j-i+1}
$$

Multiplying by $10^{N-j}$ both sides

$$
10^{N-j} \cdot g(i,j) = 
10^{N-j} \cdot f(j) -
10^{N-(i-1)} \cdot f(i-1)
$$

Let $h(i) = 10^{N-i} \cdot f(i)$. Then:

$$
10^{N-j} \cdot g(i,j) = h(j) - h(i-1)
$$

If $K$ is coprime with 10 (i.e $gcd(K, 10) = 1$),
then:

$$
K \mid (10^{N-j} \cdot g(i,j))
\Rightarrow
K \mid g(i,j)
$$

(Here $A \mid B$ denotes $A$ divides $B$)

We have reduced this problem to finding number of
pairs $(i,j)$ ($i < j$) such that $h(i) = h(j)$.

***

### Problem agc-003-B

You have some cards. Each card is labelled with
integers from $1$ to $N$. There are $A_i$ cards 
which are labelled with $i$.

Two cards can form a pair if absolute difference
of their labels is no more than 1. 
Each card can be used in at most one pair.

Find the maximum number of pairs that can be formed.

We have: $1 \le N \le 10^5$, $0 \le A_i \le 10^9$

**Solution**

Let us obtain an upperbound for number of pairs.
It is fairly obvious that the number of pairs 
can not exceed
$\lfloor{\frac{S}{2}}\rfloor$ where:

$$
S = \sum_{1 \le i \le N}{A_i}
$$

This upper bound is not very strict. Consider the
case
```
3 0 3 0 3
```
Here it is clear that only 3 pairs can be formed
while the upper bound is
$\lfloor\frac{9}{2}\rfloor = 4$ pairs.

One may claim, by observing this example, that 
the separation by 0s cause some loss. Exploring
further, we make some solid claims.

(*Definition 1*) **Component**: A range $(p, q)$
such that $A_i \neq 0$ $\forall$ $p \le i \le q$
and this range can't be extended on either side
without violating the condition or exceeding the
boundary ($1$ or $N$)

By this definition it is clear that we can divide
the whole range into components.

(*Claim 1*): *A pair can only be formed from cards
of the same component.*

*Proof*: By definition of component, two components
can't be adjacent. Thus, absolute difference of labels
would exceed 1 if a pair is formed from two different
components.

(*Claim 2*): *For each component, we can form 
number of pairs equal to the max number of pairs
possible.*

*Proof*: The max number of pairs possible for 
a component $(p, q)$ is
$\lfloor\frac{S_{p, q}}{2}\rfloor$ where

$$
S_{p, q} = 
\sum_{p \le i \le q}{A_i}
$$

We prove the claim by the following algorithm
```
let ans = 0
for i = p to q
  let k = floor(A[i] / 2)
  ans = ans + k
  A[i] = A[i] - 2 * k
  if (i < q)
    ans = ans + 1
    A[i + 1] = A[i + 1] - 1
```

For each index we form the maximum number of pairs
of possible. If a card is left, we pair it with
a card from next index (which is guaranteed to exist).
In this way only the last index will have a card left
in case of overall sum being odd.

***

### Problem agc-014-B

Suppose you have a tree with $N$ vertices.
Initially all edges hold value $0$.

You have been given
$M$ queries of form $(a_i, b_i)$ for $i = 1 \ldots N$
where $1 \le a_i, b_i \le N$ $(a_i \neq b_i)$

For a query $(u, v)$ add $1$ to each edge in path from
$u$ to $v$.

It is given that the following condition holds:

Condition:
After executing all the queries it holds that
each edge has an even value.

You have to check whether this is possible,
i.e., there exists a tree for which the given
condition holds.

**Solution**

Let $f(u)$ be the sum of values of edges adjacent
to vertex $u$. After executing all queries,
$f(u)$ is even for $u = 1,\ldots,N$ (since value
of all adjacent edges would be even).

Let us see the effect of executing query
$(u, v)$. Let the path from $u$ to $v$ be
$<x_1, \ldots, x_k>$ with $x_1 = u$ and $x_k = v$.

We will have:

$f(x_i) \leftarrow f(x_i) + 2$ for
$i = 2, \ldots, (k-1)$ (adding one to edges
$(x_{i-1}, x_i)$ and $(x_i, x_{i+1})$).

$f(x_i) \leftarrow f(x_i) + 1$ for
$i = 1, k$

If there are odd number of occurences
of vertex $u$ in the queries, then there will
be at least one edge having odd value 
adjacent to $u$.

Otherwise, we can show that atleast one tree 
satisfies the given condition.

Consider vertex $1$ as root and vertices
$2,\ldots,N$ as its children.
Edge $(1, u)$ corresponds to vertex $u$ because
each occurence of $u$ in a query will lead to
increase in value of this edge. Since there
will be even number of occurences of $u$ in the
queries, the value of this edge will be even.

In fact, any tree will satisfy the given condition.
Let $r$ be the root. For a query $(a, b)$, the
following are equivalent:
- Adding 1 to each edge on path from $a$ to $b$
- Adding 1 to each edge on path from $a$ to $r$ and
$r$ to $b$

This is because the path from root to the lowest 
common ancestor will occur in paths $a$ to $r$ and
$r$ to $b$ and thus $2$ will be added to each edge
in that path.

***

*That is it.*
