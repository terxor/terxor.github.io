---
author: terxor
title: Segment Trees - 2
category: cp
katex: true
---

*Suggested reading: Segment Trees - 1*

We begin with some more analysis of a segment tree.
(assume that ${ql},{qr}$ are set globally as discussed earlier)
Consider the following method $f$:

```cpp
void f (int z) {
  if (in_range(z)) { g(z); return; }
  if (out_range(z)) { h(z); return; }
  f(2 * z);
  f(2 * z + 1);
}
```

We state the following without proof (will write proof later):
- For any query range ${ql},{qr}$: upon calling $f(z)$, at most $4h_z - 1$ total
  calls of $f$ will be made (for $h_z \gt 0$).
- If the intersection of query range $({ql},{qr})$ with $(l_z,r_z)$ is of the
  form $(x, r_z)$ or $(l_z,x)$, then upon calling $f(z)$, at most $2h_z+1$ total
  calls of $f$ will made.
  (where $h_z$ is the height of the subtree rooted at node $z$. We have $h_1 = k$).

From the above statements, we have: 
- Upon calling $f(1)$, at most $O(k)$ total calls of $f$ will be made.
- There are $O(k)$ nodes associated with any query range $(q_l,q_r)$ meaning
that each such node $z$ will satisfy $q_l \le l_z \le r_z\le q_r$. Let's call
such nodes in-range nodes. These are the nodes for which $g$ will be called in
the method described above.

${query}$ is a specific case of the $f$ described above. Thus, for any range
query, $O(k)$ nodes are visited. We say a node $z$ is visited if there is a
method call of ${query(z)}$ in the execution. For correctness of the result of a
range query, each of the $O(k)$ in-range nodes should hold the correct value,
i.e, for each such node $z$, $s_z$ should hold the correct value (according to
the definition). Values of other nodes, including visited but out-range nodes,
will not affect the solution for this range query.

We now look at a variation of the previous problem (in part 1)
Given an integer array $a$ of size $n$ (elements being $a_0,\ldots,a_{n-1}$) you
have to answer $q$ queries which can be of the following types:
- Sum of range: Given $l, r$ return $a_l+\ldots+a_r$
- Add in range: Given $l, r, x$ add $x$ to $a_l,\ldots,a_r$

with constraints:
- $1 \le a_i, x \le 10^9$
- $1 \le n, q \le 10^6$
- $0 \le l \le r \le n - 1$

In this problem we have range updates rather than point updates.
We may apply point update $r-l+1$ times for the range update query but it will
be very inefficient.

We now introduce the concept of lazy updates or lazy propagation in segment
trees.
Lazy propagation means that the update applied to the range is **not immediately
applied** to all involved nodes (for the given target range), but it is applied on
an **on-demand basis**. When an update is yet to be applied on a node, we call it
pending update. If a node has a pending update, it may not hold the correct
value (for node $z$, $s_z$ may not be correct). Moreover, if a node has a
pending update, all the nodes in its subtree may not hold the correct value.

We call the process of applying a pending update to a node as 'pushing the update'. It involves:
- Computing the correct value for the node based on the update.
- Clearing the update on that node.
- Setting pending updates on children nodes (applicable for non-leaf nodes)

For example: If there is a pending update on a node $z$ for adding $x$ to each
target element included in the range of that node, then to push this update:

> Note that $u_z$ denotes the pending update on node $z$ ($u_z = 0$ in this case
means no pending update)

```cpp
void push (int z) {
  // Computing the correct value
  s[z] += (r[z] - l[z] + 1) * u[z];
 
  // Setting pending updates on children
  if (z < start) {
    u[2 * z] += u[z];
    u[2 * z + 1] += u[z];
  }

  // Clearing the update
  u[z] = 0;
}
```

A node $z$ holds the correct value if there is no pending update on $z$ or any
of its parents. Thus, if we wish to obtain the correct value of a node, we must
push updates of all of its parents (starting from the root). Let's modify
${query}$ method to support this:

```cpp
int query (int z) {
  push(z);
  if (in_range(z)) return s[z];
  if (out_range(z)) return 0;
  return combine(query(2 * z), query(2 * z + 1));
}
```

Note that we call push on each call to query. Although calling ${push}$ will be
useless if $z$ is an out-range node, it simplifies the implementation.

For update, first let's analyse the trivial update.

```cpp
void update_trivial (int z) {
  if (out_range(z)) return;
  if (l[z] == r[z]) { s[z] += qv; return; }
  update_trivial(2 * z);
  update_trivial(2 * z + 1);
  pull(z);
}
```

The trivial update method reaches out to each leaf for update which makes it
inefficient. Notice that the ${pull(z)}$ call is there to set the correct $s_z$
value for a node $z$ after its children have been updated.

The lazy update method sets pending updates to all in-range nodes related to the
query range (which are $O(k)$ in number)

```cpp
void update (int z) {
  push(z);
  if (in_range(z)) { u[z] += qv; push(z); return; }
  if (out_range(z)) return;
  update(2 * z);
  update(2 * z + 1);
  pull(z);
}
```

We know that lazy update (the method above) is better than trivial update
because it stops going deeper upon reaching in-range nodes. The push call on
start of each update and query method calls simplifies the analysis - it makes
the visited part of the tree same as a segment tree without lazy propagation. A
mathematically rigorous analysis is avoided here. However, the concept is
intuitive - correctness is obtained on-demand.

The combined implementation for segment tree with lazy updates is available
[here](https://github.com/terxor/cp-common/blob/main/impl/data-structures/lazy-seg.h).

*That is it.*