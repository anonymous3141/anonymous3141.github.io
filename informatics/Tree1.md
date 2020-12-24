{% include head.html %}
# Algorithms regarding to Trees
Trees is one of the greatest areas in graph theory covered in informatics contests, and is increasing in popularity rapidly. In this blog we cover some tree algorithms.
### Notes
- Let $\mathcal{T}$ denote a Tree
- Familiarity is assumed with fundamental algorithms of trees like LCA and important principles like Euler tour and diameter.
## Two Lemmas on the diameter
1. If node sets $X$ and $Y$ are diameters of a tree, then $X\cap Y\neq \emptyset$.
2. For any node $v\in \mathcal{T}$, the furthest node $u$ from $v$ is the endpoint of at least 1 diameter path.

We Prove these lemmas Here.
### Proof (1):
We proceed constructively. Consider a midpoint of one of the diameters X (i.e a middle node in the path). If this midpoint is not also a midpoint of the other diameter then we can form a longer path by connecting one of the halves of X to a part of Y. We leave the details to the reader. This midpoint is called the *centre*.

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
- APIO 2020 Fun tour  
- IOI 2015 Towns

## Centres of the Tree
There are several important points in every tree

**Centroid:** This node minimises the sum of distances to every other node in the tree, and also has many other useful properties

**Centre of tree:** Minimises the depth of the tree if rooted at it. Found by taking the midpoint of the diameter. A tree has one or two centres.

We will not in depth cover the Centre, but focus on Centroid.

**Property of Centroid:**
In addition to minimising the sum of distances, the centroid splits the tree into subtrees, none of which have more nodes than half the side of the tree. 

**Proof:** Suppose otherwise, then if we shift the centroid towards subtree with the majority of the nodes then we reduce distance sum. We can use this of thus construct the centroid by starting at a leaf and repeatedly walking in a direction that minimises distance sum.

**Applications:**
The main application for centroids is *centroid decomposition*. Effectively, we find the centroid, remove it and recursively apply the process onto the resulting subtrees. It is equivalent to divide and conquer on tree, and facilitates many effective algorithms for interactive and data structure problems.

**Example:** Maintain weighted tree with edge updates, after each update find diameter length.  

**Solution:** Consider the set of trees $\mathcal{S}$ produced by the centroid decomposition (i.e the original tree, the trees produced by the deletion of the first centroid, the second, etc). The total size of these trees is $O(NlogN)$ by the divide and conquer procedure. In this procedure, the intact diameter path must pass through the centroid of at least one of these trees. Thus by rooting each of the trees in $\mathcal{S}$ at the centroid of each tree, we only need to consider the paths passing through the root of each centroid tree and take the maximum overall. This can be maintained with lazy proprogation segment tree. When updates happen, we update the centroid trees of the $O(logN)$ centroid trees containing that edge. 

This demonstrates the key idea of centroid decomposition, that every path passes through a centroid in some centroid tree. 
Centroid decomposition has other applications too, such as:
- Binary search on Tree
- Setting up a tree for pairing style problems where a majority is undesirable

**Problems to ponder**
Dynamic Diameter (CEOI20)
JOI2020 Capital City

## Virtual Tree
The whole concept of virtual tree centres around the following fact:

**Fact:** Given a tree $\mathcal{T}$ and $K$ paths on this tree, the union of these paths' edges results in a subgraph with $O(K)$ nodes having a degree other than 2.
**Proof of fact:** Let Proceeding inductively, lets add the paths one by one to the subgraph, under the assumption that the subgraph takes the shape of a tree (if the end subgraph is a tree, then such a sequence must exist). Each path crosses the subgraph at most twice (otherwise there would be a cycle) by entering then leaving the subgraph. Thus all we do by adding the path is adding two branches to the tree. We leave the reader to formalise this into a proof.

**Applications:** Given problems that involve paths on a tree, virtual tree lets us consider subsets of these paths in isolation in good time. Let us consider the following problem: 

**Problem:** Given tree of $N$ nodes, answer $Q$ queries of the forms; 
- Given a set of paths on this tree, deduce the diameter of the subgraph resulting from the union of these paths. 
- Given a set of points on this tree, deduce the furtherest pair of points apart

The sum of the number of paths and points over all queries is $K$.

**Solution:** To Achieve $O((N+K)logN)$ complexity the problem reduces down really to building the virtual trees (forest) per query, compressing sequences of degree two "important" (not degree 2) nodes on the trees down to weighted paths, then finally running the standard diameter algorithm on the tree. Thus the problem boils down to constructing virtual trees efficiently. Now we discuss the construction of the virtual forest henceforth in 2 approaches.

**Approach 1: for set of points**

This elegantly applies LCA. Rooting the tree at 1, we note that any node which does not have degree 2 is the LCA of two of the nodes. We consider the following claim..  
*Lemma:* Consider the nodes $v_1 ... v_n$ in euler ordering (taking first occurrence of each label). Then for any $(x,y)$ there exist $z$ that $LCA(v_x,v_y)=LCA(v_z,v_{z+1})$. 
*Proof:* Map the tree into an array of integers, where the index key is the position on euler ordering and index value is the depth of the node in that position. Then $v_l=LCA(v_x,v_y)$ corresponds to the node that is the result of $rangeminquery(pos(v_x), pos(v_y))$. Consider the nodes immediately to left and right of $v_l$ on the euler ordering. Then it follows that the LCA of these nodes is also $v_l$ by property of RMQ, as these nodes are necessarily enclosed by the interval $(pos(v_x), pos(v_y))$ as required.

Thus, we can effortlessly find the important nodes of the virtual tree, and it remains to link them up. We can do this by sweeping nodes by depth from high to low. Maintain a map keyed by euler order. When a node is encountered delete the nodes in the map contained by the interval it covers and set them as its children, being conscious of the edge lengths (take difference in depth). Then insert the node itself into the map. 
INSERT FIGURE
  
**Approach 2: for set of paths**

If we merely build the virtual tree for the set of path endpoints, then it must be that the actual virtual forest is a subset of the edges of this virtual tree. We simply must mark the edges which are relevant within this virtual tree. Thus we have the problem of being given the virtual tree of n nodes and calculating the union of paths on the tree in $\tilde{O}(n)$. Lets assume all paths go from ancestor (head) to descendant (tail) (otherwise split each path in two at the LCA). Euler traverse the tree, and when the head of a path is encountered insert the tail to a set that is keyed by euler tour. To decide whether an edge is on we merely have to check if there is an active node within the subtree of the edge. Now each path's tail is only inserted and deleted once, so complexity is $O(nlogn)$. 
INSERT FIGURE

**Summary Pseudocode**
```
INPUT paths $\{(u_1,v_1) ... (u_n, v_n)\}$, tree $\mathcal{T}$
```

## Greedy algorithms and small merge
-Job scheduling
-Pushing stuff up through subtree
## Sibling dp

## Shaving off leaves and applications to edge matching
Pair programming

## HLD

## General Problems
- Baltic OI 2020 Day 2 Village
- ACIO 2020 Open Contest Black Panthers
- ACIO 2020 Open Contest Vibe Check
- CEOI 2020 Spring cleaning
- CEOI 2019 Magic Tree
- BOI 2015 Network
- IOI 2019 Practice Contest Job Scheduling
- IOI 2018 Highway Tolls
- IOI 2013 Dream
- IOI 2011 Race
