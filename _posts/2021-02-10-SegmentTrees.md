---
excerpt: on advanced usage of segment trees
---

{% include head.html %}
# Advanced Usage of Segment Trees

Segment Trees are some of the most versatile data structures in informatics. We examine creative usages of segment trees in this blog that are essential to a informatics skillset.

### Assumed Knowledge
We assume the reader is familiar with basic 1D segment trees and lazy propagation as well as walking the segment tree. Many sites on the internet, such as on codeforces, offer excellent treatments of the basics.

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
 
This shows the endless possibilities segment trees offer. The latter exercise especially, the bracket matching segment tree, has its idea applicable to many similar problems

**Exercise 1:** Given an array of  $N$ nonnegative integers (even after operations), support the two operations in $O(logN)$ time:
 - Output the length of the maximum length subarray containing only the number 0
 -  Add a potentially negative integer to every value within a range.

Solutions will be provided at the end
	 

## Part II: Applications of Lazy create segment trees

###  As an very powerful set and/or priority queue
While not as powerful as self balancing binary trees, lazy create segment trees offer almost as powerful, far simpler, and in most cases much faster implementations of ordered data.

Consider a sequence of numbers $A=a_1 ... a_N$. We maintain them in a segtree built on the frequency array of $A$ (i.e each leaf $i$ maintaining the number of entries in $A$ with that value. This allows the data to be implicitly ordered. We can do operations such as
 - what is kth largest element in the set?
 - what is the sum of elements with value between $l$ and $r$ respectively?
 -  how many elements with value in a range?

and all the while allowing numerous update forms like
 - Add many elements of a certain value
 - change the value of an element
 - Operations on value ranges

and so on. 

For example in the ACIO problem [pancakes](https://acio-olympiad.github.io/2021/pancakes.pdf) the problem boils down to implementing the data structure on a set of integers $A[1] ... A[N]$:
```
Ask(k):
	int Sum := 0
	for i = 1 ... N:
		Sum += min(A[i], k)
	return(Sum)

Upd(i, v):
    A[i] := v
``` 
but quickly.
We achieve this through the lazy create segment tree as before, leaving the reader to fill in the details. (Maintaining the quantities sum and number of things in a numerical value range is sufficient)

## Part III: Applications of segment tree in dynamic programming

Other than using segment trees to optimise recursions, segment trees can be used in many ways in dp.
### Range and/or frontier dps
Frontier dps are quite fundamental to informatics. Simple examples of problems that can solved with this technique include
 - Given an array of integers we select some of these integers. Find the maximum sum of selected integers if no two adjacent entries can be selected
 - Given a $5\times N$ grid with some squares blocked out find the number of paths from (1,1) to (5,N) (say modulo $10^9+7$)

However, there will be times where it is required to solve problems like those aforementioned, but with updates (say the value of an array entry or whether a cell is blocked gets changed).

In these cases, we can embed our dps into our segment trees. In ordinary dps, our dp will look like $dp[position][frontier]$. For instance in the first example it would look like $dp[pos][havetakenlast]$. To convert to a updatable segment tree variant we simply turn a 1 sided frontier into a 2 sided frontier, that is, for the range maintained by a segment tree node, we maintain the frontier on both its left and right ends. 

In the case of the first example we maintain
 - Max sum if leftmost element not chosen
 - Max sum if rightmost element not chosen
 - Max sum if both leftmost and rightmost element not chosen
 - Max sum without any restrictions

In general for such approaches segment tree nodes contain $O(F^2)$ values, where $F$ is the number of states on the frontier.

Pushing up involves matching 2 frontiers (left, right) and (left', right') , and applying them to become (left, right'). The complexity for this is often $O(F^3)$ as merging calculating each value on the merged node generally takes $O(F)$. 

Thus, we can convert a $O(NF)$ frontier dp into a     
$O((N+QlogN)F^3)$ version that supports updates.

This frontier merging draws parallels to matrix multiplication. The reader is left to tackle example 2, and will realise what this parallel is.

### Maintaining dp arrays with segment tree
In cases where dp recursions are simple, two dimensional dps can be optimised to $O(NlogN)$ by maintaining the second dimension in a segment tree, then using range updates to maintain the dp values when transitioning the first dimension. 

We will examine the following problem:

**NOI 2020 D1 T2:** Given a tree, rooted at node 1, on $N\leq10^5$ nodes each edge is coloured red or blue. $M\leq10^5$ constraints are given in the form of ancestor-descendant node pairs $(u_i,v_i)$, indicating that at least 1 edge in the path from $u_i$ to $v_i$ is coloured blue. The problem requires us to count the number of valid colourings modulo a prime.

**Solution:** 
To clean things up a bit, for each edge we may ignore all but one of the constraints ending at that edge, keeping the one with the lowest start point for it is the most stringent (satisfying it satisfies all other constraints). For a edge lets denote the startpoint of the most stringent constraint ending at that edge $s(i)$

An $O(n^2)$ dp is not difficult to devise. At a subtree $dp[i][j]$ denotes the number of ways to colour the edges in the subtree of edge $i$ when the lowest ancestor edge that had been coloured blue is edge $j$. Recursion involves examining the edge $i$, and determining whether or not to colour it (of course, we may be forced to colour it if a constraint ending at edge $i$ is not satisfied).



While this solution scores a credible number of points, we desire for more. To do this, we write the recursion in full:

$$dp[i][j] =\begin{cases}
\prod_{chd} dp[chd][j]+ \prod_{chd} dp[chd][i]& \text{Can assign i red}\\
\prod_{chd} dp[chd][i] & \text{Otherwise}\\
\end{cases}$$ 
For the base case, if there is no children to an edge (it connects a leaf node), then $$dp[i][j] =\begin{cases}
2& \text{Can assign i red}\\
1 & \text{Otherwise}\\
\end{cases}$$ 
(it is clear why this is so, as if we cannot assign red we must assign blue).

Now for a last blue ancestor $j$ we can assign edge $i$ to be red iff $dep(j)\geq dep(s(i))$ (as otherwise the blue edge $j$ does not satisfy the constraint at the current node). 

Observing the dp transition, it seems that the recursion is remarkably well disciplined, exhibiting both range properties (the values of $j$ for instance). We thus maintain the $j$ dimension with a lazy create lazy propagate segment tree. Denote the segment tree for subtree of edge $i$ as $S_i$ (thus $S_{i,j}=dp[i][j]$). A segment tree node stores if the range it governs has a uniform value, and the value if so.

Now suppose we have the segments of all the children of a node $v$. We want to combine them into the segment for $v$ i.e $S_{chd1} ... S_{chdk} \to S_v$.

Then we initialise all the nodes in $S_v$ to value $\prod_{chd} dp[chd][i]$ with a single range add operation. The difficult part now is adding $\prod_{chd} dp[chd][j]$ (which is different for each $j$) for consecutive ranges. While this may seem intractable, two observations come to our rescue.

 **Observation 1**: If the subtree has $x$ nodes there are $O(x)$ different values in the segment tree.
 **Proof:** By induction. In the base case of the dp there are 2 values within the dp, 1 and 2. Consider merging 2 subtrees rooted at $A$ and $B$ of $a$ and $b$ nodes respectively into the subtree at $i$. Then due to the merging procedure, if $dp[A][j]=dp[A][k]$ and $dp[B][j]=dp[B][k]$ and $j$ and $k$ allow edge $i$ to be painted both red and blue, then $dp[i][j]=dp[i][k]$. Proceeding by this logic, the subtree has at most $a+b+1$ change points (the +1 due to the threshold of the ancestor), and the induction falls out nicely.

**Observation 2**: We can small merge.
Consider merging $S_a$, $S_b$ into $S_i$ (WLOG $a$ is the bigger subtree). Then if $S_a$ and $S_b$ both have uniform values in the range $[l ... r]$, then we can merge $S_b$ into $S_a$ by having $S_{a,i}$ multiplied by $S_{b,i}$. Using a lazy create segment tree allows us to do this work in short order. This technique is called "mergable segment tree", and is a form of small merge. Remember that small merge is a key tool for tree problems, optimising greedy and dp alike.

Using small merge, we can merge the children's segment trees into $S_i$ in time $O(Slog^2S)$ where $S$ is the size of the subtree (segment trees accounting the first log, and small merge accounting for the second)

The total complexity is $O(M+Nlog^2N)$ using these tools.

The author spent 4 hours of his time writing this sample code that unfortunately MLE's but contains the spirit of the correct solution.
```cpp
#include <iostream>
#include <vector>
#define MAXN 500005
#define MOD 998244353
#define LL long long
using namespace std;
struct Node {LL val, lazy1, lazy2; //lazy1 is add, lazy2 is multiply
            int l,r;}; //always multiply before adding

void modadd(LL &a, LL b) {
    a = (a + b) % MOD;
}

void modmult(LL &a, LL b) {
    a = (a*b) % MOD;
}

class LazyCreate {
public:
    vector <Node> ST; //keyed by depth of ancestor
    //if node has no children all values in range has node's value
    void addnode(int cur) {
        ST.push_back(Node {ST[cur].val,0,1,-1,-1});
        ST.push_back(Node {ST[cur].val,0,1,-1,-1});
        ST[cur].l = ST.size() - 2;
        ST[cur].r = ST.size() - 1;
    }

    void pushdown(int cur) {
        modmult(ST[cur].val,ST[cur].lazy2);
        modadd(ST[cur].val, ST[cur].lazy1);
        if (ST[cur].l != -1) { //tags must coexist at some point
            modmult(ST[ST[cur].l].lazy1, ST[cur].lazy2);
            modmult(ST[ST[cur].r].lazy1, ST[cur].lazy2);
            modmult(ST[ST[cur].l].lazy2, ST[cur].lazy2);
            modmult(ST[ST[cur].r].lazy2, ST[cur].lazy2);
            modadd(ST[ST[cur].l].lazy1, ST[cur].lazy1);
            modadd(ST[ST[cur].r].lazy1, ST[cur].lazy1);
        }
        ST[cur].lazy1 = 0;
        ST[cur].lazy2 = 1;
    }

    void upd(LL val, bool b, int lo, int hi, int l, int r, int cur) {
        //range add val
        pushdown(cur);
        if (lo > r || hi < l) {return;}
        if (lo <= l && hi >= r) {
            if (b) { //add
                modadd(ST[cur].lazy1,val);
            } else {
                modmult(ST[cur].lazy2,val);
            }
            pushdown(cur);
            return;
        }
        if (ST[cur].l == -1) {addnode(cur);}
        int mid = (l + r) >> 1;
        upd(val, b, lo, hi, l, mid, ST[cur].l);
        upd(val, b, lo, hi, mid+1, r, ST[cur].r);
    }

    LL getval(int ind, int l, int r, int cur) {
        pushdown(cur);
        if (ST[cur].l == -1) {return(ST[cur].val);}
        int mid = (l + r) >> 1;
        if (ind > mid) {return(getval(ind, mid+1, r, ST[cur].r));}
        else {return(getval(ind, l, mid, ST[cur].l)); }
    }

    void init() {
        ST.push_back(Node {0,0,1,-1,-1});
    }
} MST[MAXN]; //lmao mergable segtree

int N, M;
int max_dep, S[MAXN], big[MAXN], par[MAXN], dep[MAXN], sz[MAXN], chd[MAXN]; //tightest constraint, parent, depth, subtree size
vector <int> adj[MAXN];

void dfs(int u, int prev) {
    par[u] = prev;
    dep[u] = dep[prev] + 1;
    sz[u] = 1;
    max_dep = max(max_dep, dep[prev]);
    for (int v: adj[u]) {
        if (v == prev) {continue;}
        dfs(v, u);
        sz[u] += sz[v];
    }
}

void mergeTree(int t1, int t2, int cur1, int cur2) { //lazy create
    //merge t1 into t2 recursively
    MST[t1].pushdown(cur1);
    MST[t2].pushdown(cur2);
    if (MST[t1].ST[cur1].l == -1) {
        modmult(MST[t2].ST[cur2].lazy2, MST[t1].ST[cur1].val);
        MST[t2].pushdown(cur2);
    } else {
        if (MST[t2].ST[cur2].l == -1) {
            MST[t2].addnode(cur2); //wanna AC? in this case link t1's node to t2 and lazy t2 instead
        }
        mergeTree(t1, t2, MST[t1].ST[cur1].l, MST[t2].ST[cur2].l);
        mergeTree(t1, t2, MST[t1].ST[cur1].r, MST[t2].ST[cur2].r);
    }
}

void dfs2(int u) {
    //do the dp, Segtree stores at i what is best if last taken edge was at height i
    //edge whose parent has depth i has depth i too
    big[u] = 0;
    int thresh = dep[S[u]]; //can take both red and blue for thresh ... N
    LL prod = 1;
    for (int v: adj[u]) {
        if (v == par[u]) {continue;}
        dfs2(v);
        modmult(prod, MST[big[v]].getval(dep[par[u]], 0, max_dep, 0)); //hope no +-1
    }
    for (int v: adj[u]) {
        if (v == par[u]) {continue;}
        if (sz[big[v]] > sz[big[u]]) {
            swap(big[v], big[u]);
        }
        if (big[v]) {mergeTree(big[v], big[u], 0, 0);}
    }

    if (!big[u]) {
        big[u] = u;
        MST[big[u]].upd(1,1,0,max_dep,0,max_dep,0); //base case
    }
    MST[big[u]].upd(0, 0, 0, thresh-1, 0, max_dep, 0); //nuke [0...thresh-1] by multiplying by 0
    MST[big[u]].upd(prod,1, 0, max_dep, 0, max_dep, 0); //watch +- 1 for thresh, colouring edge u-> par[u] is universal option

}
int main() {
    //freopen("fatein.txt","r",stdin);
    scanf("%d",&N);
    LL a = 1;
    for (int i=1; i<N; i++) {
        int u,v;
        scanf("%d %d",&u,&v);
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    dfs(1,0);
    S[1] = 1; //nice WLOG
    for (int i=0; i<=N; i++) {
        MST[i].init();
    }

    scanf("%d",&M);
    for (int i=0; i<M; i++) {
        int a,b;
        scanf("%d %d",&a,&b);
        if (dep[a] > dep[b]) {swap(a,b);} //a is shallower
        if (dep[S[b]] < dep[a]) { //if a is deeper relax
            S[b] = a;
        }
    }

    dfs2(1);
    printf("%lld", MST[big[1]].getval(0,0,max_dep,0));
}
```
## Part IV: Putting things in segment trees

### Sets and priority queues
Rev up the dimensionality
### Vectors
The merge sort Tree

### Sorted Stacks

Fancy footwork that may shave off a log relative to using Sets instead (which can almost always be applied when Sorted Stacks can), but we won't cover these as Set is usually more clear cut)
### Convex Hull tricks
A big load of nope, but unfortunately the best solution often for computational geometry.
## Part V: Persistence
Applications to 2D range query, walking for kth largest
## Part VI: Divide and conquer methodology
Refer to my camp lecture ppt for further details. We detail how we can use segment tree methodology in data structure problems such as Dynamic Graph Connectivity offline

## Part VII: Segment trees on Tree
Segment trees can be applied with great effect on trees. However, the star of the show is not the segment tree itself, but the ways of reducing a tree down to an array problem. Techniques include
 - euler tour sweep and segment tree on depth (path problems)
 - build tree on euler tour array (subtree problem)
 - HLD (more difficult path problems)

Refer to the next section for discussion regarding sweeps

## Part VIII: Segment Trees and sweep

## Part IX: Solutions
**Exercise 1:** For each node maintain the greatest length subarray containing only the minimum element in the range, as well as the suffix and prefix length of the range which has the minimum element, as well as the value of the minimum element. Push-up can be done relatively simply.
