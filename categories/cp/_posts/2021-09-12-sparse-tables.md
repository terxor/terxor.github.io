---
author: terxor
title: Sparse Tables
category: cp
katex: true
---

A sparse table is yet another data structure useful
for operations on ranges.

Given elements $A_0, A_1, \ldots, A_{n-1}$ and binary function
$f$ which is associative (sum for example), sparse table can help find value of
$f$ applied over a range $l$ to $r$ i.e., $f(A_l, A_{l+1},
\ldots, A_r) $ where $0 \leq l < r \leq n-1$ (for $l = r$, the
required value is $A_l$ itself in most cases)

Very common examples of $f$ are sum, product, minimum, maximum
and bitwise operations such as AND, OR, XOR.

> A binary function $f(a, b)$ is said to be **associative** if $f(a,
  f(b, c)) = f(f(a, b), c)$ which means that the grouping of
  arguments does not matter. Thus, we can use a simpler notation
  while applying this function over a sequence of elements, for
  example: Applying $f$ on $a, b, c$ can be represented as $f(a,
  b, c)$.

> A binary function $f(a, b)$ is said to be **commutative** if $f(a,
  b) = f(b, a)$ which means that the order of arguments does not
  matter.

> A binary function $f(a, b)$ is said to be **idempotent** if $f(a, a)
  = a$.

Take $f(a, b) = min(a, b)$ for example: it is associative,
commutative and idempotent which essentially means that min can
be treated as a *variable argument function (atleast one
argument) which ignores repeated (non-distinct) arguments*.
Suppose we wish to find minimum of 1, 4, 2, 5, 4. Then we could
write min(1, 4, 2, 5, 4) or equivalently min(1, 2, 4, 5)

### Idea and definition

A sparse table for $n$ elements takes $O(n\lg n)$ time to
initialize and $O(n\lg n)$ space throughout its use. It can
answer queries (result of applying associative function $f$ over
a given range) in
- $O(1)$ if $f$ is also commutative and idempotent
- $O(\lg n)$ otherwise

However, the elements must not change throughout its
use, i.e., it can only be used for queries on \textbf{static
data}.

The main idea of sparse table is to store results of $f$ applied
to ranges whose sizes are powers of 2. Let there be $n$ elements
$A_0, A_1, \ldots, A_{n-1}$ and an associative function $f$. Let
$k$ be the minimum integer satisfying $2^k > n$. This implies $k
= \lfloor \lg n \rfloor + 1$

The sparse table can be represented as 2D array (0-indexed) of
dimensions $n \cdot k$ whose definition is as follows:

$$
h(i, j) =
\begin{cases}
f(h(i, j - 1), h(i + 2^{j-1}, j - 1)) &\text{ if $j > 0$ and $i
+ 2^j - 1 < n$ } \\
A_i &\text{ if $j = 0$ }
\end{cases}
$$

This definition implies that $h(i, j) = f(A_i, A_{i+1}, \ldots,
A_{i+2^j-1})$


> Note: $h(i, j)$ is not defined for $j > 0$ and $i + 2^j - 1 \geq n$.
This is because range of size $2^j$ starting with index $i$ can
not exist in this case. This also explains the name
*sparse* table. There is a fairly large number of cells in the 2D
array which are not used.

Each cell of this 2D array can be computed in $O(1)$ given that
$f(a, b)$ can be computed in $O(1)$. Thus, the time to build the
sparse table is $O(n \cdot k) = O(n\lg n)$

**Querying when $f$ is associative, commutative and idempotent**

To find the value of $f(A_l, \ldots, A_r)$, perform the
following steps:
- Let $m = r - l + 1$ (range size). Let $z$ be the maximum integer
  such that $2^z \leq m$.
- Let $x = h(l, z)$ and $y = h(r - 2^z + 1, z)$. Return
  $f(x, y)$.

These steps take $O(1)$ time (assuming $f(a, b)$ can be computed
in $O(1)$).

Let us prove the correctness informally. $x$ and $y$ are the
results of $f$ being applied to leftmost and rightmost $2^z$
elements of the range (respectively). Since $2^z + 2^z = 2^{z +
1} > m$ (because $z$ is the maximum integer
such that $2^z \leq m$), it is guaranteed that the whole range
is covered. However, there will be some overlap between the
leftmost $2^z$ and rightmost $2^z$ elements but it will not
affect the result because $f$ is commutative and idempotent.

*Implementation available [here](https://github.com/terxor/cp-common/blob/main/impl/data-structures/sparse-table-1.cpp)*

**Querying when $f$ is associative but not commutative or idempotent**

Since $f$ is not commutative or idempotent, we can not longer
have an overlap while combining results of multiple ranges. We
can compute $f(A_l, \ldots, A_r)$ by the following recursive
definition:

In the above definition:
- $m = r + l - 1$ (range size) and
- $z$ is the maximum integer such that $2^z \leq m$

$$
query(l, r) = 
\begin{cases}
h(l, z) & \text{if $2^z = m$} \\
f(h(l, z), query(l + 2^z, r)) & \text{if $2^z < m$}
\end{cases}
$$

This way we combine the results of non-overlapping ranges (by
repeatedly chipping off highest power of 2 not greater than the range
size from the left of the range).

$query(l, r)$ takes atmost $O(\lg n)$ recursive calls because $z$,
the highest power of 2 which is not greater than the range
size, will decrease by atleast 1 in each recursive call to
$query$.

*Implementation available [here](https://github.com/terxor/cp-common/blob/main/impl/data-structures/sparse-table-2.cpp)*

***
