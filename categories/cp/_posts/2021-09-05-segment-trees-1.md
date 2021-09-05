---
author: terxor
title: Segment Trees - 1
category: cp
katex: true
---

A segment tree is a versatile data structure which can be used to solve many
programming problems. In this discussion we will represent a segment tree as a
perfect binary tree. Thus, some knowledge of representing a perfect binary tree
as an array is required. 

We state the following without proof: 
(assume $0$-indexing for the array)
- To represent a perfect binary tree of height $k$, an array of size $2^{k+1} - 1$ is required. However, we will not use index $0$ in the array which means we would require size $2^{k+1}$.
- The index of root node is $1$. 
- Node with index $z$ will have its left and right child at indices $2z$ and $2z+1$ respectively, if this node is not a leaf.
- Node with index $z$ will have its parent at index $\lfloor z/2 \rfloor$, if this node is not the root.
- There are $2^k$ leaves with indices $2^k,\ldots,2^{k+1}-1$.
- Level order traversal (left to right on each level then moving to the next level, starting from root) can be performed by iterating through indices $1,\ldots,2^{k+1}-1$.

***

Let's begin with a basic example - range sum queries.

Given array $a$ of size $n$ (elements being $a_0,\ldots,a_{n-1}$) you have to answer $q$ queries of form $l, r$ which have to be answered with the sum $a_l+\ldots+a_r$

Constraints:
- $1 \le n, q \le 10^6$
- $0 \le l \le r \le n-1$

If working with ranges in a segment tree (which is quite common), we have to keep the following points in mind:
- Each node has a range associated with it. Node $z$ has range $(l_z, r_z)$ of target indices. To differentiate between the indices of nodes (in the array which represents the segment tree) from the indices given in the problem (queries of form $l, r$), we term the latter 'target' indices.
- A leaf represents a range of size $1$ (single element), which means $l_z = r_z$. The leftmost leaf (having index $2^k$, where $k$ is the height) represents range $(0, 0)$, i.e, target index $0$. Leaf $i$ represents range $(i, i)$ for $0 \le i \le 2^k-1$ (the leftmost leaf is leaf $0$, leaf index increases as we move towards right)
- For a node which is not a leaf, we have: $l_z = l_{2z}$ and $r_z = r_{2z+1}$. By this definition, the root node covers the range $(0,2^k-1)$ of target indices.
- Each target index is included in exactly $k$ nodes (where $k$ is the height of the tree)

We now know the indices of the nodes and target ranges they represent. The
value/element a node holds is dependent on the problem and is defined according.

For the given problem, each node will hold an integer which is the sum of all
elements in the target range represented by that node.

We define an integer array $s$ of size $2^{k+1}$ ($s_0,\ldots,s_{2^{k+1}-1}$)
which will be used for representing the segment tree.

If we have $n$ target indices to cover ($0,\ldots,n-1$), we have to find the
minimum $k$ such that $2^k \ge n$. There might be some (possibly zero) nodes
which cover some target range disjoint with $(0,n-1)$. Such nodes will hold some
dummy value which will not affect the solution (in this case, having $s_z = 0$
for some node $z$ which represents a target range disjoint to $(0,n-1)$).

We have arrays $s, l, r$ whose meanings are already discussed. For convenience,
we have a variable 'start' which is the index of the first leaf. Leaf $i$ has
index ${start} + i$. Since ${start}=2^k$, the size of these arrays is $2 \cdot
start$.

```cpp
vector<int> s, l, r;
int n, k = 0, start;
while ((1 << k) < n) k++;
start = (1 << k);
s.assign(2 * start, 0);
l.assign(2 * start, 0);
r.assign(2 * start, 0);
```

Segment trees are suited for associative functions, for instance: addition,
which is required in this problem. We call this associative function 'combine'
in the implementation.

```cpp
int combine (int a, int b) { return a + b; }
```

The value for a node $z$ can be obtained by applying 'combine' on its children
nodes. We call this operation 'pull'.

```cpp
void pull (int z) { s[z] = combine(s[2 * z], s[2 * z + 1]); }
```

We need to set the value of the leaves according to the target values. In the implementation, we also set $l$ and $r$ values. Then, we 'build' the segment tree by moving up the levels. Note that $l$ and $r$ values are also computed.

```cpp
// Setting leaf values
for (int i = 0; i < start; i++) {
  s[start + i] = (i < n ? a[i] : 0);
  l[start + i] = r[start + i] = i;
}

void build() {
  for (int i = start - 1; i >= 1; i--) {
    pull(i); l[i] = l[2 * i]; r[i] = r[2 * i + 1];
  }
}
```

For convenience, we have global variables for query range $(ql,qr)$. Also, we have functions for checking:

- Whether the target range represented by node $z$ is disjoint with the query range.
- Whether the target range represented by node $z$ lies completely within the query range.

```cpp
int ql, qr;
inline bool out_range (int z) { return (ql > r[z] || qr < l[z]); }
inline bool in_range (int z) { return (ql <= l[z] && r[z] <= qr); }
```

The 'query' method takes a node index as an argument and returns the sum of the
range of target indices which is part of the query range. If target range of the
node lies completely within the query range, the already computed sum $s_z$ is
returned. The query method is called with argument $1$ (root node) and the total
number of recursive calls is $O(k)$. This can be informally proved as follows:
After the first recursive call, the subsequent calls will have intersecting
target ranges (with query range) of the form $(x, r_z)$ or $(l_z, x)$ which
means a base case is guaranteed to occur in one of the two further recursive
calls (to left and right child).

```cpp
int query (int z) {
  if (in_range(z)) return s[z];
  if (out_range(z)) return 0;
  return combine(query(2 * z), query(2 * z + 1));
}
```

The given problem has a much easier solution (prefix sum array) which is much
more efficient. However, if there were queries which required adding some $x$ to
$a_i$ then the prefix sum solution would fail. The queries which involve update
to a single target element are called 'point' updates. Segment trees allow
efficient updating by exploiting the fact the each target index is included in
$k$ nodes. Thus, any change in some leaf node would require changing $k$ nodes
(including the leaf itself).

```cpp
void update (int i, int x) {
  int z = start + i;
  s[z] += x;
  while (z / 2) { z /= 2; pull(z); }
}
```

An implementation involving ideas from this post can be found [here](https://github.com/terxor/cp-common/blob/main/impl/data-structures/seg-tree.h).

*That is it for part 1.*