{% include head2.html %}
{% include head.html %}
# Advanced Usage of Segment Trees

Segment Trees are some of the most versatile data structures in informatics. We examine creative usages of segment trees in this blog that are essential to a informatics skillset.

### Assumed Knowledge
We assume the reader is familiar with basic 1D segment trees and lazy propagation. Many sites on the internet, such as on codeforces, offer excellent treatments of the basics.

## Part I: Pushup functions
Segment Trees should be used flexibly. In the words of my predecessors who have taught me this, *one should not try to fit a problem into the mould of a common segment tree, but should fit a segment tree if possible into the problem*.

This sounds rather abstract, and I did not fully appreciate this for a long time. To unpack this statement, note that segment trees, in its essence, consist of nodes storing packets of data, and a function to merge these packets of data into another data packet of the same form.

While these packets are often just single integers, and the merge function (called a pushup by many, including me) are often as simple as an addition or maximum, they can be so much more. We consider some examples of this.

**Problem 1:** You have graph with $N \leq 100$ nodes and $M\leq 10^5$ edges numbered 1 through $M$. We need to answer $Q\leq 10^5$ queries of form $l,r$: 
 - Using only edges $l$ through $r$, how many connected components in the graph?
 
**Solution:**  We build segment tree on the edges. However, in each node we store spanning forests of the graph from the edges in the range. Merging nodes can be done in $O(n)$. Thus using the normal segment tree query methodology (each query returns a node effectively) we can process a query in $O(NlogM)$ time. Precalculation can be done in $O(MN)$ time (as there are $M-1$ merges at $O(N)$ per merge).

**Problem 2:** Given a bracket sequence of length $N$ support the two operations:
 - Change a bracket within the sequence
 - Query whether the bracket substring from $l ... r$ is perfectly matched.

**Solution:** We implore the reader to consider the problem first before reading this solution, and to keep his mind open to the possibilities of segment trees as from before. 

For a segment tree node let us store:
 - The number of unmatched ")" brackets 
 - The number of unmatched "(" brackets
 
 in the range. To combine left and right children, we match as many (unmatched) "(" brackets in the left child to as many ")" brackets in the right child as possible. A range can be fully matched if and only if there are no unmatched brackets.
 
This shows the endless possibilities segment trees offer.

**Exercise 1:** Given an array of  $N$ nonnegative integers (even after operations), support the two operations in $O(logN)$ time:
 - Output the length of the maximum length subarray containing only the number 0
 -  Add a potentially negative integer to every value within a range.

Solutions will be provided at the end
	 

## Part II: Applications of Lazy create segment trees

###  As an very powerful set and/or priority queue
USACO 2019 Jan Plat P1, ACIO Pancakes
###  Mergeable segment trees
Small Merge

### Lazy dps
Use segment tree to assist the mapping of dp states. For instance, the LIS.
## Part III: Putting things in segment trees

### Sets and priority queues
Rev up the dimensionality
### Vectors
The merge sort Tree

### Range and/or frontier dps
They often remind us of matrices

### Sorted Stacks

## Part IV: Persistence
Applications to 2D range query, walking
## Part V: Divide and conquer methodology
Refer to my camp lecture ppt for further details. We detail how we can use segment tree methodology in data structure problems such as Dynamic Graph Connectivity offline

## Part VI: Solutions
**Exercise 1:** For each node maintain the greatest length subarray containing only the minimum element in the range, as well as the suffix and prefix length of the range which has the minimum element, as well as the value of the minimum element. Push-up can be done relatively simply.
