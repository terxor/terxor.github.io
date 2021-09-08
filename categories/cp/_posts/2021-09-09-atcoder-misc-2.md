---
author: terxor
title: Review of misc. problems - 2
category: cp
katex: true
---

*Source of these problems is [atcoder](http://atcoder.jp)*

### Problem abc-189-C

You are given a histogram $N$ columns whose values
are $A_1,\ldots,A_N$ (all integers). Find the area of the largest
rectangle that can be formed within the histogram.

We have $1 \le N \le 10^5$, $0 \le A_i \le 10^9$.

**Solution**

Let's define *good range* for some index $i$ as $[l_i, r_i]$
satisfying:
  - $1 \le l_i \le i \le r_i \le N$
  - $l_i$ is minimum possible
  - $r_i$ is maximum possible
  - $A_i = min \lbrace A_j \mid l_i \le j \le r_i \rbrace $.

In simpler terms, the max we can go left or right without getting
a lower value than $A_i$ (and not crossing the boundary, of course)

Let the optimal solution be range $[L, R]$ with minimum height $H$ resulting
in area $(R - L + 1) \cdot H$. 
In this range we have minimum element $H$.  Let $i$ ($L \le i \le R$) be such
an index that $A_i = H$. Then it holds that $L = l_i$ and $R = r_i$ (by the definition
of $[l_i, r_i]$).

Thus we can obtain the answer by checking for each element $A_i$ being
minimum in a range (and the maximum size range is its *good range*).
The answer is given by:

$$
\max_{1 \le i \le N} \left( A_i \cdot (r_i - l_i + 1) \right)
$$

To obtain $l_i, r_i$ efficiently:
- Keep a set of indices $S = 1, \ldots, N$. Add $0$ and $N+1$ for convenience.
- Let $x_1, \ldots, x_N$ denote non-increasing order of elements.
- Process indices in the order $x_1, \ldots, x_N$.
- When processing $x_i$:
  - Find maximum element lesser than $x_i$ in $S$. Let it be $u$.
  - Find minimum element greater than $x_i$ in $S$. Let it be $v$.
  - Then we have $l_i = u + 1$ and $r_i = v - 1$.
  - Update answer and remove $x_i$ from the $S$.

Sample implementation of these operations:
```cpp
// after preparing set 's' and array 'x'
int ans = 0;
for (int i = 0; i < n; i++) {
  auto itr = s.lower_bound(x[i] + 1);
  int v = *itr - 1;
  itr = s.upper_bound(x[i] - 1);
  int u = *(--itr) + 1;
  ans = max(ans, a[x[i]] * (v - u + 1));
  s.erase(s.find(x[i]));
}
cout << ans << '\n';
```

***

### Problem m_solutions2019_D

You have a tree with $N$ vertices $1,\ldots,N$. You are given:
- $N-1$ edges $(a_i, b_i)$ for $i = 1,\ldots,N-1$
- $N$ values $c_1,\ldots,c_N$ 

You have to
assign a value to each vertex ().

Assign values $c_1,\ldots,c_N$ to vertices $1,\ldots,N$ maximizing:

$$
\sum_{1 \le i \le N-1}{min(v_{a_i}, v_{b_i})}
$$

where $v_i$ is the value assigned to vertex $i$.
Note that $c_i$ should be assigned to exactly one vertex for $i = 1\ldots N$

> In other words, assign values to vertices such that if for
  each edge we write the minimum of the values of
  endpoints, then the sum of values of each edge is maximized.

**Solution**

Assume that $c_1 \le \ldots \le c_N$.

Consider the following pseudocode 
which generates a sorted list of values obtained
from each edge (i.e, min of values of end points)

Assume that value $c_i$ has been assigned to
vertex $v_i$ for all $i = 1,\ldots,N$
($v_1,\ldots,v_N$ is a permutation of $1,\ldots,N$)

```
l = []
for i = 1 to N-1
  let p = v[i]
  for j in adj[p]
    let q = v[j]
    remove edge (p, q)
    l.append(c[i])
return l
```

The above method works as follows:

- Once an edge is processed, it is removed. Therefore,
each edge is processed exactly once.
- Vertices are processed in non-decreasing order of their
assigned values. While processing any edge $(v_i, v_j)$
through vertex $v_i$, it holds that $i < j$ and therefore
$c_i \le c_j$. Thus, $min(c_i, c_j) = c_i$ is appended
to the list for this edge.
- It is fairly obvious why the output list is sorted.
- There is no need to run the iteration for $i=N$
because it will have no adjacent edges at that point.

*Claim 1*: In the for loop of above pseudocode, the following holds 
for all $i$, $1 \le i \le N-1$:
at the end of iteration $i$, the list $l$ has size atleast
$i$ 

*Proof*:

We will prove this claim by induction on $i$.

For $i=1$ it holds true because vertex $1$ has
atleast one adjacent edge (which will be processed
in that iteration)

Suppose it holds true for $i=k-1$, then the list
has atleast $k-1$ elements at the end of iteration $i-1$. We will show 
that it holds true for $i=k$

Suppose that, at the beginning of iteration $k$, the list has
exactly $k-1$ elements (because if it has greater than $k-1$
elements, then the condition will be true for $i=k$)

If $v_i$ has at least one adjacent edge, then value
$c_i$ will be appended at least once in this iteration.
Otherwise, $v_i$ will be isolated and there will be
$N-i$ remaining edges and $N-i$ vertices (excluding $v_i$)
indicating that the structure is not a tree or a forest
contradicting the fact that given structure was a tree.

*Claim 2*: In an optimal assignment, the sum of values
on edges can't exceed $c_1+ \ldots + c_{N-1}$ 

*Proof*:

Suppose the sorted list of edge values is $l$.
This list will be the same as that obtained from
the above pseudocode. By claim 1 it directly follows
that $l_i \le c_i$. Thus,

$$
\sum_{1 \le i \le N-1}{l_i} \le
\sum_{1 \le i \le N-1}{c_i}
$$

Now, we present a method to obtain the value
$c_1 + \ldots + c_{N-1}$. Consider an arbitrary
vertex to be the root. Assign value $c_N$ to the root.
Assign values $c_{N-1}$ down to $c_1$ according to
BFS order of processing the vertices. It will hold
that parent of any vertex will not have a lesser value.

*Alternate proof of claim 2*

Assume $c_1 \ge \ldots \ge c_N$. The claim now becomes:
In an optimal assignment, the sum of values on edges
can't exceed $c_2 + \ldots + c_N$

First of all, any subgraph of $k$ edges will have atleast
$k+1$ vertices and thus will have min edge value $\le c_{k+1}$
(because max $k+1$ values are $c_1, \ldots, c_{k+1}$)

Consider an optimal solution. Suppose the edges have values
$x_1 \ge \ldots \ge x_{N-1}$.

Initially we have $N-1$ edges.
It holds that minimum edge value $x_{N-1} \le c_N$ (by above statement).
Remove this edge. Now repeat. If we have $i$ edges, it holds that
the minimum edge has value $x_i \le c_{i+1}$.

Therefore, $x_i \le c_{i+1}$ $\forall$ $i, 1 \le i \le N-1$ and:

$$
  (x_1 + \ldots + x_{N-1}) \le (c_2 + \ldots + c_N)
$$

***

### Problem agc-032-B

You are given an integer $N$. Build an undirected
graph with $N$ vertices with indices $1$ to $N$
that satisfies the following two conditions:

- The graph is simple and connected.
- There exists an integer $S$ such that, for every
vertex, the sum of the indices of the vertices
adjacent to that vertex is $S$.

It can be proved that at least one such graph
exists under the constraints of this problem.

We have $3 \le N \le 100$.

**Solution**

Suppose we partition the vertices into
$K \ge 2$ sets, with sum of labels of vertices in each
set being the same, say $x$.

Then for each vertex, we
connect it with every other vertex except
for those which are in the same set as that
vertex. This leads to the sum (of labels of adjacent vertices) 
of each vertex to be $(K-1) \cdot x$.

For even number of vertices, i.e., $N = 2P$,
we can partition the vertices into $P$ (set $i$
having vertices $i$ and $2P-i+1$, sum being
$2P+1$)

For odd number of vertices, i.e., $N = 2P+1$,
we can proceed in the same manner as for even
number of vertices while considering an
extra set containing only vertex $2P+1$.

For implementation, one can simply add
an edge between each pair of vertices
except for those whose labels add up to $2P + 1$
where $P = \lfloor \frac{N}{2} \rfloor$.

***

### Problem arc_111_B

We have $N$ cards numbered $1$ to $N$
Each side of each card has a color represented by a positive integer.
One side of Card $i$ has a color $a_i$, and the other side has a color $b_i$

For each card, you can choose which side shows up. Find the maximum possible
number of different colors showing up.

**Solution**

This can be modelled to a graph problem:
Consider each color as a vertex and each card as an edge.

> You are given a graph. For each edge, only one
> end (vertex) can be activated. Activating an
> already activated vertex is allowed and it will
> have no effect. At most how many vertices can
> be activated?

- Observation 1 ($O_1$): For a tree, all except one
  vertices can be activated ($N$ vertices, $N-1$
  edges) by starting with an arbitrary vertex
  as root and activating children and repeating
  with subtrees. The root of the tree will not
  be activated in this way. 

  - Observation 2 ($O_2$): For a connected component which is
  not a tree ($N$ vertices, $M$ edges, $M \ge N$) all
  vertices can be activated. How? first
  activate a cycle, which is guaranteed to exist Then,
  from each vertex of cycle perform DFS/BFS and
  activate the traversal tree by method stated
  in $O_1$. Self loops and multiple edges can be
  handled in this way.

by $O_1$ and $O_2$, each connected component allows
$min(N_e, N_v)$ activated vertices where $N_e, N_v$ are
number of edges and number of vertices in that component, respectively.

$O_2$ can be stated more simply as: find a
spanning tree. There will be atleast one edge not
a part of the spanning tree. Activate any
endpoint of this edge and assume that endpoint
as root. Then by $O_1$, all other vertices can be
activated.

***

*That is it.*
