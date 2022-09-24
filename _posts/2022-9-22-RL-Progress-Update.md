
---
excerpt: An update on my connect four RL project
---

{% include head.html %}

# RL Project Progress Update

It has been a while since I last touched my RL project on Connect Four from Janurary, but it has always been at the back of my mind because the results I obtained in Janurary were rather unsatisfactory, the agent wasn't able to learn much at all!

I've recently been able to come back to it thanks to the summer break I will use this blog post to detail what I've done.

## Experiment Aim

Build the strongest Connect4 AI you can via RL and no previous knowledge engineered in

## Some themes to what I've done differently

- **Good code**: After a summer software internship, I realised my old experiment code was absolutely horrible. There were dependencies everywhere and you couldn't debug it as you can't tell what went wrong (and trust me a lot of things do go wrong in RL). Now I've changed the repo to a much more structured and modular OOP design with non-zero amounts of unit tests
- **Start simply**: Last time I was too quick to try apply all different kinds of RL algorithms. Now I'm working exclusively in the framework of state value based policies,  and trained with the simplest approach, monte carlo learning
- **Tune more hyperparameters**: The modular structure makes it easy for me to see what experiment choices I made, and now makes it easier me to change these around. Examples include NN architecture, epsilon decay schedules and optimizer choice. It really does make a world of difference!

## Setting up the training pipeline
1. Build and test the connect4 enviroment. As many games will need to be played, a highly optimised bitmask implementation was prepared. Build and test enviroment to play against AI (to understand what it is playing)
2. Start implementing some strategies. 
	- Pure random
	- Play randomly unless you can win next turn (OneMoveLookAhead)
	- Monte Carlo Tree Search with customisable rollout and number of simulations per move
		- You  might wonder why I decided to make MCTS so early on. It's a surprise tool that will help us out later, but its also quite fun to play against. 
3. Implement the neural network architecture. I propose two distinct architectures:
	- Arch1: A $4\times 4$ convolutional layer followed by 2 dense layers and an output layer
	- Arch2: A $4\times 4$ followed by $2\times 2$ convolutional layer, followed by 1 dense layer and an output layer
	- Nonlinearity was initially ReLU (we'll talk about this later), and my friend helpfully suggested Tanh as the final activation.
		- This lets me map position values to [-1,1] and amplify decisions that are close to 0
	- My compute is regrettably very limited, so I'll have to be stingy-ish with the model size
4. Implement the trainer class. The optimization is done by pytorch and the RL method is the naive monte carlo one:
	- Play a game against an opponent. Suppose the states are $s_1 ... s_n$ and final result is $r$. Then we'll train the value function against the input pairs $(s_i, \gamma^{n-i}r)$ where $\gamma=0.9$ is chosen as the decay factor. Mean squared error is the metric I chose to optimize against
	- Exploratory behavior also had to be programmed in. I chose the simple $\epsilon$-greedy method, with $\epsilon$ scheduled some way
	- Initially, the opponent is fixed to be the OneMoveLookAhead() function. Later, this would be modified into a more advanced training pipeline similar to how alphaGo was trained.
	- I also intend to implement proper bootstrapping (SARSA style) at some point but I think that it is not the highest priority right now, I have enough choices to deal with at present
5. Integrate my neural net into Negamax and MCTS to see what it can do

## Experiments and experiences messing with all sorts of cooked problems

Even after copius amounts of unit tests, it was still very difficult to get anything working. I'll detail my observations and some of the more interesting things I dealt with while trying to train the value function.

This doesn't include anything I met while implementing the enviroment (for instance I had to deal with ``np.randomchoice`` casting the ``pyInt``s  in my list of valid moves into ``int32`` and that causing the bitboard to overflow when it was passed into it), which itself was an very interesting experience.

### Phase 1: Training against Fixed agent

The most pressing thing on the agenda was to get a neural net that wasn't awful. This is actually pretty hard to do. Some parameters that I needed to tune included:
- The epsilon schedule
- The optimizer
- The network choice (arch1 or arch2)
- The size of the network 
- Which agent to train against

First, I chose some baseline parameters  that usually are decent from literature
- epsilon scheduled to decay exponentially, initially from 1 to a minimum level of 0.01 after $80\%$ of input games have been run. 
- Run 10000 training games (rip compute)
- Adam optimizer with learning rate 0.001
- Train NN against MCTS with 50 rollouts per turn

And subject to this a $2\times 2$ grid search with 'small' and 'large' network sizes (large is twice the width of small) as well as both architectures were made. 

While the learning was pleasingly quick, with the neural net taking a few minutes to train, none of the methods demonstrated significant learning. After looking at neural network predictions, it was realised neural network was always outputting the same value irregardless of input, and this was as we were running into the classical 'dead neuron issue', where a neuron is rendered dead due to its linear weights being badly set. 

Researching solutions, I decided to switch relu to LeakyRelu (leaky = 0.01 gradient) and also decrease learning rate to 0.0001. 

After rerunning the experiment in this setting, I still wasn't successful in getting any of the 4 networks to learn anything. I figured then it was because MCTS(50) was too good in some sense. My intuition is that you don't want to teach a person math starting from say complex analysis rather than addition, so it is better to start easy.

So I switched to training against OneMoveLookAhead, and finally I got some results. First, Arch2 failed miserably, and so did Arch1-small. They never managed to win more than 50% of time (in 100 games) against OneMoveLookAhead, but Arch1-big managed to win 86 in 100 games which is very impressive. This corresponds to arc1-64-exponential.pth in my parameter list. 

Realising that neural net size mattered (think GPT2 vs GPT3 I suppose), I upped the size again to 100 conv filters & nodes in dense layer, and retrained, achieving similar results to the 64 node case (i.e Arch1-big). Then, I played against the AI and analysed its behavior. Here is what I observed
- The thing is actually pretty smart sometimes. It has figured out that playing in the middle is good,  knows to cluster its tokens together and also knows to complete 4 in a row
-  It plays much worse as player 2 than player 1 (for some reason) 
- It is really bad at defending, and lets the opponent complete 4 in a row pretty often rather than blocking it. Perhaps it takes more time to learn this.

Lastly, I tested the TD training method, but it didn't work very well (it seems tricky to use bootstrapping well), so I gave up quickly.

But overall, I'll call this a success. It was great to finally see signs of intelligence after a lot of work. This corresponds to the arc1-100-exponential.pth parameter file

### Phase 2: Self-Playing

Now that we can beat OneMoveLookAhead() fairly reliably, it has come time to see how much better we can do. 

So I decided to implement a self-play training procedure, in style somewhat reminiscint of alphaGo: (I created a branch in the repo for this called self-play). For ease of intepretation, Adam was switched to good old SGD.

```
CANDIDATE_OPPONENTS = [SOME BASELINES]
REPEAT N times:
	INITIALISE CURRENT AGENT FROM PREVIOUS AGENT PARAMS
	FOR M LEARNING ITERATIONS
		DECAY EPSILON
		USE NAIVE MONTE-CARLO LEARNING:
			PLAY CURRENT AGENT AGAINST RANDOM PREVIOUS CANDIDATE

	DECAY LEARNING RATE BY HALF
	ADD CURRENT AGENT TO OPPONENT LIST IF IT WINS >40% OF TIME AGAINST LAST ITERATION OF AGENT
```

After implementing this, I trained with N=10, M=10000. It turned out that gradients were exploding (the predictions of all boards were being forced towards 1 so I increased L2 regularization and the value of MIN_EPSILON = 0.05 (it seems that the agent learns more easily with a high-ish epsilon). 

Training with the new setting, the first few iterations each managed to beat the previous agent around 90~95% of the time, seeing improvements until roughly the 6th iteration (beat 60% of time, see section on significance)

Notably, this agent managed to beat the NN developed in Phase 1 in 90 out of 100 test games, demonstrating the viability of self-play (but it only wins 26 out of 100 against MCTS-50).

### Phase 3: Milking the Neural Net

This is still in my self-play branch. So we've got a decent neural net now. There's several ways we can use it to boost the strength of pre-existing mechanisms. It seems that the strength of the neural net is in playing positionally (it knows to play in the middle and set up clusters of tiles) rather than tactically (it is very bad at blocking etc), so integrating it with a search mechanism is the obvious way forward.

The three methods I had in mind were:

1. Using it as the MCTS rollout policy. This is pretty expensive computationally so I'll forget about it for the moment
2. Using it to aid the MCTS move selection, alphaGo style, where we do $V^*(s)=\lambda \text{rollout}(s)+(1-\lambda)v_\theta(s)$. Our previous approach is equivalent to $\lambda = 1$.
3. Using it to evaluate negamax positions

This is where my prior MCTS implementation comes in handy, and we'll use the model selection procedure detailed below.

We'll pairwise compare 5 agents using the significance testing mechanism detailed in the next section:
- MCTS-300 with $\lambda = 0.25, 0.5, 0.75, 1$
- Negamax with search depth 3 moves and leaf evaluated by the neural network

It turned out, that compared to the base line $\lambda = 1$ player, choosing $\lambda = 0.25$ or using negamax was worse, with MCTS winning 60% and 75% of time respectively.

On the other hand $\lambda = 0.75$ won 168 out of 300 games against $\lambda=1$ (the experiment was ran thrice on this particular occasion), which is quite nontrivial (Z-score = 2.17) but still below our criteria for significance. I suspect the neural network does in fact improve the quality of play (it certainly played very strongly against me and beat me most of the times) but not by much. This could be due to the neural network still not being precise enough.

On a more positive note, when the number of rollouts is limited to 20, then we do see the neural net providing stastically significant (by the skin of its teeth) boost in strength, with the neural net equipped MCTS (with $\lambda = 0.5$) winning 63 games out of 100 against a vanilla MCTS (so our neural net does help if the quality of the rollout value is less reliable).

### Significance testing for agent selection

To compare 2 agents, I get them to play 100 games against each other (randomising who gets white/black). The Null hypothesis is that they're equally strong so we can model the results (#wins in 100 games) as $W \sim Bi(N=100,P=0.5)$, noting that draws are pretty rare (we'll just count them as 0.5 points). We'll use a two tailed test with significance level 0.01 (as for all we know the new agent could be a lot worse, and also since we're running a lot of these tests so a higher threshold is needed to avoid p-hacking), so we'll seek the threshold corresponding to about 2.58 standard deviations above mean.

### Conclusion

The final agent involves MCTS, with an epsilon greedy tree policy, OneMoveLookAhead as rollout policy, and combined with using our CNN to return evaluations with $\lambda=0.5$. With 300 rollout simulations per turn, the agent is capable of beating myself consistently. Of course, there's still a lot that I haven't explored due to constraints in programming time / compute.


### Topics worth investigating

Despite pleasingly nontrivial performance being attained, there is
still a lot more I need to investigate though, and I hope I can revisit this sometime in the future. Some things I can look into:
- Representation of the board (I'm currently using a 6 by 7 by 2 input. Perhaps I should represent tokens as 1 or -1 rather than using a second dimension)
- Different learning algorithms: I'm currently using very naive stuff. Can we do better with more advanced RL algorithms?
- Different network designs
- Using UCB/Better rollout policy for MCTS etc

