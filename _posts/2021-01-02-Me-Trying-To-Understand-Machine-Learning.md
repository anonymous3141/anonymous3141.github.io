---
excerpt: Some notes made by me in my first attempt to do Machine Learning.
---
{% include head.html %}
## Lol I really can't be stuffed typing up more of my notes rip I guess
Also rip formatting (idk even if what I wrote is right lol)

## Definitions I guess
**Artificial intelligence** - anything a computer does thats mildly smart: shortest path problem etc
**Machine learning** - subset of AI. The computer builds models of training instances of a problem which is then applied to solve other instances of problem.

## Types of ML
- **Supervised:** Program is given input and output pairs and learns the model this way.
  - **Classification problem:** The output domain is discrete (e.g the set {0,1}). Problems include digit recognition
  - **Regression problem:** The output is $$\mathbb{R}^k$$. Problems include many physics things like experimentally finding friction coefficient
- **Reinforced learning:** Program is given input, and feedback in form of reward function based on its output. I suppose Neural Nets are a form of this.
- **Unsupervised learning:** Program is given inputs but not expected outputs, and is left to make sense of patterns in data.
  - **Clustering** is an example of this
  
## Big ideas in ML

**Representation of data:** 
So apparently the way the data is fed into the program is important (for instance entering a picture of man in a murder investigation isn't as helpful as just telling who he is), and manipulations on data is important to getting palatable trends (think log-log graphs in physics). This is where humans and deep learning comes in.


**Deep learning (in context of Neural Net):** 
Allows a NN to learn a good representation of data through multiple perceptron layers


**Humans:** 
Has intuition about whats useful

## Basic algorithms in ML that are not Neural Nets
### Least squares
It solves the regression problem of modelling a set of points $$\{(x_1,y_1) ... (x_N,y_N)\}$$ with a degree $$n$$ polynomial $$P(x) = \sum_{i=0}^{n} a_i x^i$$ where $$n << N$$ (otherwise we **overfit**). We aim to make the model accurate by minimising $$\sum_{i=1}^n (P(x_i)-y_i)^2$$.

We find the optimal polynomial by linear algebra. So consider a matrix 

$$
A=  \begin{bmatrix}
    1 & x_1 & x_1^2 & ... & x_1^n \\
    1 & x_2 & x_2^2 & ... & x_2^n \\
    ... \\
    1 & x_N & x_N^2 & ... & x_N^n 
  \end{bmatrix}
$$

and vector 

$$
b =\begin{bmatrix}
y_1\\
y_2\\
...\\
y_N\\
\end{bmatrix}
$$

The problem is equivalent to trying to find the optimal vector

$$
x =\begin{bmatrix}
a_0\\
a_1\\
...\\
a_n\\
\end{bmatrix}
$$

such that the norm

$$
||Ax-b||^2
$$

is minimised.

**Lemma 1:** The optimal $$x$$ satisfies the condition $$A^TAx=A^Tb$$.

**Proof:** Apparently there's a easy vector calculus proof but I don't know vector calculus so we're doing this dumbly. If $$C_A$$ is the columnspace of $$A$$ then essentially we're just taking $$x=proj_{C_A}  b$$. As we all know, $$C_A \perp b-Ax$$ (by definition of projection or think about Gram-Schmidt I guess) so we have for any $$v\in C_A$$ that $$<c,b-Ax>=0$$ where $$<>$$ denotes dot product. By writing $$v=Au$$ for arbitary vectors $$u$$ we have $$<Au, b-Ax> = 0\iff u^{T} A^{T} b = u^{T} A^{T} Ax$$. Now let $$k = A^TAx-A^Tb$$ and consider $$u^{T} k$$. If \(k\neq 0\) there must exist $$u$$ that $$u^Tk\neq 0$$. We leave the reader to prove this assertion. However, it is easy to see in the light of the previous equation that it implies the Lemma.

Thus we simply compute $$x=(A^TA)^{-1} (A^Tb)$$ and we are done.

**Q:Wait what this is a machine learning algorithm?**

**A:** YES! We are deducing a **model** from a set of input points, the **training set** that allows us to predict the value for future x coordinates (say the **test/evaluation set**). Let us discuss this algorithm with some machine learning terminologies.


**Q: How good is our model?**

**A:** Depends. We choose the best model from our **hypothesis space** (the set of all possible models the algorithm may choose the apply, in this case the set of all degree $$n$$ polynomials in terms of our **loss function** (a performance metric, in this case the sum of errors squared)


**Q: How can we tune our model?**

**A:** While the parameters of our model $$x$$ is chosen by the algorithm, the value of $$n$$ is not. That is a **hyperparameter**, or  value humans choose for the algorithm experimentally and/or based on prior knowledge. While hyperparameters can be chosen by another algorithm, it is most common that us humans choose it.


**Q: How do we know our model generalises?**

**A:** Broadly speaking, we don't. A major hurdle of any machine learning algorithm is to have a model that generalises. We don't want our model to **underfit**, or fit too loosely to our training data. This could be trying to fit a linear model ($$n=1$$) to a exponential growth curve, but we don't want to have a large **generalisation gap** due to **overfitting** either. This can be due to say fitting a degree 100 polynomial in our case to a simple linear model, resulting in a curve that perfectly passes through every point by zigzaging through every point, but is nonlinear and inaccurate in reality. In other words, by choosing too large a hypothesis space (allowing $$n=100$$ degree polynomials), we overtaylor our models to the training set, thus missing the true essence of the data and sacrificing performance on general data. In fact, it is generally considered desirable to use the simplest models that fit a training dataset, as usually it is these models that generalise the best. Methods exist, which will be dicussed in the future to reinforce this idea.


### K-nearest neighbour
Perhaps the most mind numbingly dumb Machine learning algorithm I've seen to date. So suppose you have a set of labels you want to classify a point as;


**For instance:** 
 You have the height and leaf length of a bunch of plants.
 You want to classify them as carrot or potato plants.
 In this case a point is (height, length) and there are 2 labels (carrot, potato)

What this algorithm does is given a test point, examine the $$K$$ nearest points (by some distance metric such as euclidean) in training set to the test point, and determine the current label by a simple majority vote between these points. Here, $$K$$ and distance metric is a hyper-parameter. It's that simple ....

**Q: Is this algorithm guaranteed to be good?**

**A:** Hell no. In fact, the **No Free Lunch theorem** implies that when averaged over all possible data generating destributions, any machine learning algorithm has the same accuracy performance as an algorithm that makes a random guess. In sane terms, suppose that we have to predict the output of an unknown functions and are given a finite set of input-output pairs (training set) to help us. Then a data generating destribution is the probability destribution underlying the random variable that generates the inputs (and it is assumed that the same destribution generates the training and test sets). This theorem proves the intuition that no algorithm can infer a whole function from only finite input output pairs. However, by assuming properties around the function, or the way data is generated (for instance that data with certain characteristics are more likely to be generated), the NFL theorem can be sidestepped, as after all, it only considers all destributions hollistically rather than the actual underlying destribution(s). Thus it is important as humans to choose the best ML algorithms for a certain job based on assumptions (for instance this algorithm assumes that things with the same label are similar)
