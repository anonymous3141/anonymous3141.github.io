{% include head.html %}
## Algorithms regarding to Trees
Trees is one of the greatest areas in graph theory covered in informatics contests, and is increasing in popularity rapidly. In this blog we cover some tree algorithms.
### Notation
Let $\mathcal{T}$ denote a Tree
### Two Lemmas on the diameter
1. If edge sets $X$ and $Y$ are diameters of a tree, then $X\cap Y\neq \emptyset$.
2. For any node $v\in \mathcal{T}$, the furthest node $u$ from $v$ is the endpoint of at least 1 diameter path.

We Prove these lemmas Here.
# Proof (1):
We proceed by contradiction. If indeed $X\cap Y= \emptyset$ then let the endpoints of the paths $X$,$Y$ be $a,b,c,d$. 
It can be seen without too much work that one of the paths between two of $a,b,c,d$ is longer than $X$ and $Y$ respectively. 

# Proof (2):
Suppose $A,B$ are the endpoint of a diameter and WLOG $d(v,A)\leq d(v,B)\leq d(v,u)$. Rooting the tree at $v$, consider $C$ 
the lowest common ancestor LCA of $B$ and $u$. Then $d(C,B)\leq d(c,u)$ by assumption that $u$ is the furtherest node.
If the path $UV$ is disjoint from $AB$ then $d(A,u) = d(A,c) + d(c,u) \geq d(A,B)$ so $u$ is also the endpoint of a diameter. 
If the paths share an edge the logic also falls out similarly.
