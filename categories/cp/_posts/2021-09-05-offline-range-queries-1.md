---
author: terxor
title: Offline Range Queries - 1
category: cp
katex: true
---

In offline query problems, we are supposed to process a number of update query
operations before answering any retrieval queries. Consider the following two
problem statements:

1.  Given integer array $a$ of length $n$ having zeroes initially, you have to process
    $m$ queries of form:

    - $(l, r, x)$ : Add $x$ to $a_l,\ldots,a_r$

    - $x$ : Output value of $a_x$

1.  Given integer array $a$ of length $n$ having zeroes initially, you have to first
    process $m_1$ queries of form:

    - $(l, r, x)$ : Add $x$ to $a_l,\ldots,a_r$

    Then, you have to process $m_2$ queries of form:

    - $x$ : Output value of $a_x$


Here, problem 1 is an online query problem and problem 2 is an offline query problem.
It should be clear that the solution for problem 1 also solves problem 2.
However, there are simpler/more efficient solutions for offline query problems.

We can solve problem 1 using segment trees. Refer the post related to segment trees.
Can we solve problem 2 in an easier way?

***

A well known solution for this problem uses the idea of prefix sums. 
Before proceeding let's define prefix sum array of a given array.

Let $b$ be the prefix sum array of an array $a$ of length $n$.
Then,

$$
b_i = a_1 + \ldots + a_i \ \forall\ i, 1 \le i \le n
$$

For computing $b$ we simply do:
- $b_1 \leftarrow a_1$
- $b_i \leftarrow b_{i-1} + a_i$ $\forall$ $i, 2 \le i \le n$ (sequentially)

Suppose you have an array of size $6$ initially containing only zeroes.

$$
a = (0, 0, 0, 0, 0, 0)
$$

Now, let us add $3$ to $a_2$

$$
a = (0, 3, 0, 0, 0, 0)
$$

Let us look at the prefix sum array $b$ of $a$

$$
b = (0, 3, 3, 3, 3, 3)
$$

Now, this gives us an idea of how a point operation transforms into a range
operation after obtaining prefix sum array. Suppose we wanted to add $3$ in
range $[2, 4]$. We have to stop prefix sum due to $3$ from propagating further
from a particular point (here, index $4$). This is obtained by subtracting.

$$
a = (0, 3, 0, 0, -3, 0)
$$

Now, the prefix sum array $b$ of $a$

$$
b = (0, 3, 3, 3, 0, 0)
$$

This gives us an clear idea of the approach, i.e., 
for each range update query $(l, r, x)$, perform the following:
- $a_l \leftarrow a_l + x$
- $a_{r+1} \leftarrow a_{r+1} - x$

After all such operations, the final correct state of the array can be obtained
by prefix sum operation.

The above example shows a single query. How is it correct in case of multiple
queries (which might have intersecting ranges)?

This can be understood by decomposing the array into $m$ arrays (one for each
query): $p_1,\ldots,p_m$.

For each query $(l, r, x)$ the corresponding array $p_i$ will have
- $p_{i, l} = x$
- $p_{i, r+1} = -x$

Let array $q_i$ be obtained as a result of prefix sum operation on $p_i$. It
should be clear that we will have: $q_{i,j} = x$ if $l \le j \le r$ and $0$
otherwise.

So, we know that the correct array would be obtained upon first obtaining the
prefix sum arrays $q_1,\ldots,q_m$ of $p_1,\ldots,p_m$ and then for each $a_i$:

$$
a_i = q_{1,i} + \ldots + q_{m,i}
$$

In the approach above (where the arrays are not explicitly decomposed), we have
(before prefix sum operation):

$$
a_i = p_{1,i} + \ldots + p_{m,i}
$$

After the prefix sum operation:

$$
a_i = 
  p_{1,1} + \ldots + p_{m,1} + \\
  p_{1,2} + \ldots + p_{m,2} + \\
  \vdots \\
  p_{1,i} + \ldots + p_{m,i}
$$

which, in terms of $q$ becomes:

$$
a_i = q_{1,i} + \ldots + q_{m,i}
$$

This shows that the correct answer is obtained. The decomposition makes
correctness obvious. We have shown that not decomposing also results in the same
answer.

***

Now we can move to a little more advanced variation of range addition - adding
arithmetic progressions in a range. Let's say adding $x$ to range $l, r$ with
common difference $y$ i.e.,

- $a_l \leftarrow a_l + x$
- $a_{l+1} \leftarrow a_{l+1} + (x + y)$
- $a_{l+2} \leftarrow a_{l+2} + (x + 2y)$
- $\cdots$
- $a_r \leftarrow a_r + (x + (r - l)y)$

We now go back to the example above which had a length $6$ array. Instead of
adding $3$ in range $[2, 4]$, we wish to add an arithmetic progression starting
with $3$ having common difference $2$. First we add and subtract $3$ at indices
$2$ and $5$ (respectively).

$$
a = (0,3,0,0,-3,0)
$$

How to introduce the common difference? Suppose we had another array $d$ so that
while computing the prefix sum of $a$, we could also include the common
difference.

$$
d = (0,0,2,2,0,0)
$$

First, let's combine arrays $a$ and $d$ (by addition on each element)

$$
a' = (0,3,2,2,-3,0)
$$

Now, we will compute the prefix sum array $b'$

$$
b' = (0,3,5,7,4,4)
$$

Notice that we have almost got what we wanted. It seems as if we didn't subtract
a high enough value at index $5$. Had we subtracted $7$ instead of $3$

$$
a' = (0,3,2,2,-7,0) \\
b' = (0,3,5,7,0,0)
$$

Let's generalize this: To add an arithmetic progression starting with $x$ having
common difference $y$ in range $[l, r]$, we have to perform the following
operations:
- $a_l \leftarrow a_l + x$
- $a_i \leftarrow a_i + y$ for all $i, l+1 \le i \le r$
- $a_{r+1} \leftarrow a_{r+1} - (x + (r - l)y)$

The correctness can be easily proved for a single query:
Initially $a_i = 0$ for all $i, 1 \le i \le n$. Upon applying the operation
above, $a_i = 0$ for all $i, 1 \le i \lt l$ and $r+1 \lt i \le n$. Let us now
obtain the prefix sum array $b$.
- $b_i = 0$ for all $i, 1 \le i \lt l$.
- $b_l = x$ (because of the first operation)
- $b_i = x + (i - l)y$ for all $i, l+1 \le i \le r$ (because of the second operation)
- $b_i = 0$ for all $i, r+1 \le i \le n$ (because of the third operation)

To prove correctness for multiple queries, one may apply the decomposition
technique as done for the simple range additions.
Notice that operation 2 is not a constant time operation, but we can change that
by applying the concept of range addition (as described previously) to the
common difference array $d$. In the example above, we needed to add $2$ in range
$[3, 4]$. We could have done:

$$
d = (0,0,2,0,-2,0)
$$

and then by obtaining prefix sum array:

$$
d = (0,0,2,2,0,0)
$$

The point is that we can replace the second operation with the following
operations (we need to maintain an additional array $d$):

- $d_{l+1} \leftarrow d_{l+1} + y$
- $d_{r+1} \leftarrow d_{r+1} - y$

We also need some extra steps after processing the queries.

- Converting $d$ to its prefix sum array
- $a_i \leftarrow a_i + d_i$ for all $i, 1 \le i \le n$

After this, we can convert $a$ into its prefix sum array to obtain the final array.

***

Problems that may involve using this concept:
[CF819B](https://codeforces.com/contest/819/problem/B)
