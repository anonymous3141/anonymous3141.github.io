{% include head.html %}
# Algorithms regarding to Trees
Trees is one of the greatest areas in graph theory covered in informatics contests, and is increasing in popularity rapidly. In this blog we cover some tree algorithms.
### Notation
Let $\mathcal{T}$ denote a Tree
## Two Lemmas on the diameter
1. If edge sets $X$ and $Y$ are diameters of a tree, then $X\cap Y\neq \emptyset$.
2. For any node $v\in \mathcal{T}$, the furthest node $u$ from $v$ is the endpoint of at least 1 diameter path.

We Prove these lemmas Here.
### Proof (1):
We proceed by contradiction. If indeed $X\cap Y= \emptyset$ then let the endpoints of the paths $X$,$Y$ be $a,b,c,d$. 
It can be seen without too much work that one of the paths between two of $a,b,c,d$ is longer than $X$ and $Y$ respectively. 

### Proof (2):
Suppose $A,B$ are the endpoint of a diameter and WLOG $d(v,A)\leq d(v,B)\leq d(v,u)$. Rooting the tree at $v$, consider $C$ 
the lowest common ancestor LCA of $B$ and $u$. Then $d(C,B)\leq d(c,u)$ by assumption that $u$ is the furtherest node.
If the path $UV$ is disjoint from $AB$ then $d(A,u) = d(A,c) + d(c,u) \geq d(A,B)$ so $u$ is also the endpoint of a diameter. 
If the paths share an edge the logic also falls out similarly.

### Applications of the two lemmas.
Arguably, Lemma 2 is the more useful of the two. Lets consider two problems.  

**Problem 1** Module task: Given hidden tree of N nodes, you can query distance between two points. Find the length of the diameter in 2N queries.   

**Answer:** Trivial application of lemma 2.

**Problem 2** Given unweighted tree construct permutation of nodes $v_1 ... v_N$ such that $d(v_i,v_{i+1})\geq d(v_{i+1}, v_{i+2})$.  

**Answer:**  Lets find a diameter of a tree and let its endpoint be $x$. We construct the sequence by repeatedly finding the furtherest node from $x$. Observe that
at each iteration of the algorithm the diameter of the tree is nonincreasing. Further, we prove inductively that at each iteration the node we currently occupy is an endpoint of at least 1 diameter on the tree. Lets suppose that from $x$ we go to $y$. Then $xy$ is a diameter path clearly by definition. Now from $y$ we can at least guarantee a diameter of $d(x,y)-1$ by going from $y$ to the second node in the path from $x$ to $y$. If there still exist a path of length $d(x,y)$ then it must share an node with the path $xy$ by Lemma 1. As such when the tree is rerooted at $y$ the endpoints of the previous diameter become accessible to $y$ and we will jump to there instead.

**Problems to ponder:**
APIO 2020 Fun tour  
IOI 2015 Towns

## Centres of the Tree
There are several important points in every tree
**Centroid:** This node minimises the sum of distances to every other node in the tree, and also has many other useful properties
**Centre of tree:** Minimises the depth of the tree if rooted at it. Found by taking the midpoint of the diameter. A tree has one or two centres.

We will not in depth cover the Centre, but focus on Centroid.

**Properties of Centroid**
In addition to minimising the sum of distances, the centroid splits the tree into subtrees, none of which have more nodes than half the side of the tree. 

**Proof:** Suppose otherwise, then if we shift the centroid towards subtree with the majority of the nodes then we reduce distance sum. We can use this of thus construct the centroid by starting at a leaf and repeatedly walking in a direction that minimises distance sum.

**Applications:**
The main application for centroids is *centroid decomposition*. Effectively, we find the centroid, remove it and recursively apply the process onto the resulting subtrees. It is equivalent to divide and conquer on tree, and facilitates many effective algorithms for interactive and data structure problems.
**Example:** Maintain weighted tree with edge updates, after each update find diameter length.  
**Solution:** Consider the set of trees $\mathcal{S}$ produced by the centroid decomposition (i.e the original tree, the trees produced by the deletion of the first centroid, the second, etc). The total size of these trees is $O(NlogN)$ by the divide and conquer procedure. In this procedure, the intact diameter path must pass through the centroid of at least one of these trees. Thus by rooting each of the trees in $\mathcal{S}$ at the centroid of each tree, we only need to consider the paths passing through the root of each centroid tree and take the maximum overall. This can be maintained with lazy proprogation segment tree.
