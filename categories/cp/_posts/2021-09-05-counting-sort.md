---
author: terxor
title: Counting Sort
category: cp
katex: true
---

A linear time, stable sort algorithm for elements having
small integer keys.

Let the given array be $A[1 \ldots n]$
Each object ($A[1],\ldots,A[n]$) has multiple attributes
(including a key, a small integer)
For object $x$, let	
- $x.k$ be the key
- $x.attr1, x.attr2, \ldots$ be other attributes.

Further, let us assume that an object is uniquely
defined by the ordered set of all of its attributes This
assumption would help in understanding stable sort more
clearly

### Stable sort problem

- Input: Array $A[1...n]$
- Output: Array $B[1...n]$ which satisfies:
	- $B$ is a permutation of $A$
	- $B[i].k \le B[i + 1].k \ \forall\ i = 1,\ldots,n-1$
		(in other words, sorted according to k) 
	- Stable sort property, i.e,
		if $B[i].k = B[j].k$ for some $1 \le i \lt j \le n$,
		let the corresponding positions in
		original array be $p$ and $q$ (that is, $B[i] = A[p]$
		and $B[j] = A[q]$) then: $p < q$. In other words, for
		elements which compare equal, they retain their
		original order.

### Counting sort algorithm

- Let $cnt[1...M]$ be an array initialized with $0$ where
$M$ such each key $k$ satisfies $1 \le k \le M$.
Let $B[1...n]$ be an empty array

- Compute frequencies
	~~~
	for i = 1 to n
		cnt[A[i].k]++
	sum = 0
	~~~

- Compute prefix sum
	~~~
	for i = 1 to n
		sum += cnt[i]
		cnt[i] = sum
	~~~

- At this point, $cnt[A[i].k$ denotes the number of
	elements having $key \le A[i].k$, if we traverse the list
	from right to left, $cnt[A[i].k]$ actually gives the index
	of the element where it should be placed.
	~~~
	for i = n down to 1
		B[cnt[A[i].k]] = A[i]
		cnt[A[i].k]--
	~~~
	The step
	`cnt[A[i].k]--` ensures that next occurence of element
	moving from right to left with same key will be placed
	at correct index) 