## What resources am I using? 
Deep Learning with Python - Chollet
Deep Learning book - Goodfellow, Bengio, Courville
Google colab for experimentation

## Definitions I guess
**Artificial intelligence** - anything a computer does thats mildly smart: shortest path problem etc
**Machine learning** - subset of AI. The computer builds models of training instances of a problem which is then applied to solve other instances of problem.

## Types of ML
- **Supervised:** Program is given input and output pairs and learns the model this way.
  - **Classification problem:** The output domain is discrete (e.g the set {0,1}). Problems include digit recognition
  - **Regression problem:** The output is $\mathbb{R}^k$. Problems include many physics things like experimentally finding friction coefficient
- **Reinforced learning:** Program is given input, and feedback in form of reward function based on its output. I suppose Neural Nets are a form of this.
- **Unsupervised learning:** Program is given inputs but not expected outputs, and is left to make sense of patterns in data.
  - **Clustering** is an example of this
  
## Big ideas in ML
LMAO how am I supposed to know I've read about 5 hours of this.
**Representation of data:** 
So apparently the way the data is fed into the program is important (for instance entering a picture of man in a murder investigation isn't as helpful as just telling who he is), and manipulations on data is important to getting palatable trends (think log-log graphs in physics). This is where humans and deep learning comes in.
**Deep learning (in context of Neural Net):** Allows a NN to learn a good representation of data through multiple perceptron layers
**Humans:** Has intuition about whats useful

## Basic algorithms in ML that are not Neural Nets
## Least squares
It solves the regression problem of modelling a set of points $\{(x_1,y_1) ... (x_N,y_N)\}$ with a degree $n$ polynomial $P(x) = \sum_{i=0}^{n} a_i x^i$ where $n << N$ (otherwise we **overfit**). We aim to make the model accurate by minimising $\sum_{i=1}^n (P(x_i)-y_i)^2$.

We find the optimal polynomial by linear algebra. So consider a matrix $
  \begin{bmatrix}
    1 & x_1 & x_1^2 & ... & x_1^n \\
    1 & x_2 & x_2^2 & ... & x_2^n \\
    ... \\
    1 & x_N & x_N^2 & ... & x_N^n 
  \end{bmatrix}
$
