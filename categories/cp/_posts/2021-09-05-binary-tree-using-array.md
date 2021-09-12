---
author: terxor
title: Binary Tree using array
category: cp
katex: true
---

In this post we discuss how to represent a binary tree
using a linear array. We
will be concerned only with perfect binary trees.
If we can represent a perfect binary tree $T$ of
height $h$, then we can represent any binary tree $T'$ of
height $h$ by simply omitting some nodes from $T$.

### Analysis

In a perfect binary tree, what is the number of nodes on
level $x$? Define $f_x$ to be the number of nodes on level
$x$. On level $0$ we have a single node - the root. Any
other level will have nodes equal to twice of that on
previous level (because each node will have exactly two
children - the left child and the right child)

$$
f_x=
\begin{cases}
	1 &\text{if $x=0$} \\
	2\cdot f_{x-1} &\text{otherwise}
\end{cases}
$$

which simplifies to $f_x = 2^x$

Thus, a perfect binary tree of height $h$ will have a total of

$$
f_0 + f_1 + \cdots + f_{h} \\
=2^0 + 2^1 + \cdots + 2^{h} \\
=2^{h+1} - 1 \text{ nodes} 
$$

Let us assign numbers to each node starting by assigning $1$
to the root node.
- At each level, we start from the leftmost
  node on that level moving towards the rightmost node
(incrementally numbering them).
- After reaching the rightmost node of a level, we move to the next level or stop if the
current level is the last level.
- It should be clear that the
rightmost node on the last level will be numbered
$2^{h+1} - 1$ since this is the last node to be numbered.

Here is a perfect binary tree of height $3$.

![binary-tree](/assets/images/cp/binary-tree.svg)

The index of node at the start of level $k$ will $2^k$
because we have already numbered
$f_0 + f_1 + \cdots + f_{k - 1} = 2^{k} - 1$
nodes.

- Thus the nodes on level $k$ will be numbered.
  $2^{k}, 2^{k}+1, \cdots, 2^{k}+2^{k}-1$.
  The $i^{th}$ node on this level will be numbered
  $2^{k}+i-1, 1 \leq i \leq 2^{k}$
- Similarly the nodes on level $k+1$ will be numbered
  $2^{k+1}, 2^{k+1}+1, \cdots, 2^{k+1}+2^{k+1}-1$.
  The $i^{th}$ node on this level will be numbered $2^{k+1}+i-1,
  1 \leq i \leq 2^{k+1} $

What are the indices of children of $i^{th}$ node on level
$k$ - the node having index $2^{k}+i-1$?
- It is obvious that the children of this node are at level $k+1$.
- The first two nodes on level $k+1$ are the children of the first node on
  level $k$, the next two nodes on level $k+1$ are the
  children of the second node and so on.
- To find the indices
  of children of $i^{th}$ node on level $k$, we first skip
  $2\cdot(i-1)$ nodes on level $k+1$ because these are the
  children of first $(i-1)$ nodes. The next two nodes are the
  left and right child respectively.

Thus, index of left child is $2^{k+1}+(2\cdot(i-1)+1)-1$

and index of right child is $2^{k+1}+(2\cdot(i-1)+2)-1$

If we let $m=2^{k}+i-1$ then,
- Index of left child is $2 \cdot m$
- Index of right child is $2 \cdot m + 1$

> This does not hold for leaf nodes
  (which have indices $2^{h}, 2^{h}+1, \cdots, 2^{h}+(2^{h}-1)$)

What happens when $2\cdot m > 2^{h+1}-1$? 
Note that it also means $2\cdot m+1 > 2^{h+1}-1$.
Would this mean that children of this
particular node don't exist (since there are only
$2^{h+1}-1$ nodes)? Let us see

$2 \cdot m > 2^{h+1}-1$

$\Rightarrow m > \frac{2^{h+1}-1 }{2} $

What is the smallest integer $m$ for which this becomes true?

$m = \left\lfloor \frac{2^{h+1}-1}{2} \right\rfloor + 1$

$m = \left\lfloor 2^{h} - \frac{1}{2} \right\rfloor + 1$

$m = 2^{h} - 1 + 1$

$m = 2^{h}$

This is precisely the index at which we reach the last level
at which all nodes are leaf nodes.

We know that children of node with index $m$ have indices
$2\cdot m$ and $2\cdot m+1$. From this we claim that
index of parent of a node $x$, $(2 \leq x \leq 2^{h+1}-1)$ is given
by $\left\lfloor \frac{x}{2} \right\rfloor$ since 
$$ \left\lfloor \frac{2\cdot m}{2} \right\rfloor = m$$
and 
$$ \left\lfloor \frac{2\cdot m+1}{2} \right\rfloor = m$$
This generalization will be correct if there is no other
value $v$, $(2 \leq v \leq 2^{h+1}-1)$ other than $2\cdot m$ and $2\cdot m+1$ which
gives $m$ upon floored division by two and
that is indeed the case. 

### Conclusion

To represent a perfect binary tree of height $h$ in an
array, the size $N$ of the array must be $2^{h+1}$ if it is
$0$-based (the element at index $0$ will not be used).

The indices
$ 2^{h}, 2^{h}+1, \cdots, 2^{h+1} - 1$
$= \frac{N}{2}, \frac{N}{2} + 1, \cdots, N-1 $
will represent leaves.

For a node with index $m$
- The index of left child is $2\cdot m$ $\forall$ $1 \leq m < \frac{N}{2}$
- The index of right child is $2\cdot m+1$ $\forall$ $1 \leq m < \frac{N}{2}$
- Parent is $\left\lfloor \frac{m}{2} \right\rfloor$ $\forall$ $1 < m < N $

It is worth noting that this method is especially useful for
complete binary trees -- in which each level, except for the
last one, is completely filled. Representation of binary
trees having large heights and sparsely filled levels will
be inefficient using this method as a lot of memory will be
wasted.

***