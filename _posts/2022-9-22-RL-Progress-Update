---
excerpt: on advanced usage of segment trees
---

{% include head.html %}

# RL Project Progress Update

It has been a while since I last touched my RL project on Connect Four from Janurary, but it has always been at the back of my mind because the results I obtained in Janurary were rather unsatisfactory, the agent wasn't able to learn much at all!

I've recently been able to come back to it thanks to some time in the summer break and spend a few days reworking the experiments. I will use this blog post to detail some progress on my project. 

## Some common themes to what I've done differently

- After a summer software internship, I realised my old experiment code was pretty bad. It was spaghetti, dependencies everywhere and you couldn't debug it as you can't tell whats wrong. Now I've changed to a much more structured and modular design and unit tested as I wrote each module, and it works pleasingly a lot better now
- Start simply: Last time I was too quick off the bat to try apply all different kinds of RL algorithms. Now I'm working exclusively in the framework of value function policies, trained with the simplest approach, TD(1) (i.e naive monte carlo learning)
- Tune more hyperparameters: The modular structure makes it easy for me to see what experiment choices I made, and now makes it easier me to change these around. Examples include NN architecture, epsilon decay schedules and optimizer choice. It really does make a world of difference!

## Experiment Aim

Build the strongest Connect4 AI you can via RL and no previous knowledge

## Setting up the training pipeline
1. Build and test the connect4 enviroment. As many games will need to be played, a highly optimised bitmask implementation (from [1]) was prepared. Build and test enviroment to play against AI (to understand what it is playing)
2. Start implementing some strategies. 
	- Pure random
	- Play randomly unless you can win next turn (OneMoveLookAhead)
	- Monte Carlo Tree Search with customisable rollout and number of simulations per move
	- You  might wonder why I decided to make MCTS so early on. But its two fold. For one, it makes a potent agent to train against, and for second, it is a very good way to improve any policy (in fact, setting rollout policy to be OneMove LookAhead already managed to beat me which was very surprising but impressive)
3. My goal now is to train a good position evaluation function $v_\theta (x)$ and use it with the monte carlo tree search. There's two ways to do it
	- One is as the rollout policy in MCTS
	- The other is as data to augment the randomised rollout, as per the AlphaGo paper
4. Implement the neural network architecture. I propose two distinct architectures:
	- Arch1: A $4\times 4$ convolutional layer followed by 2 dense layers and an output layer
	- Arch2: A $4\times 4$ followed by $2\times 2$ convolutional layer, followed by 1 dense layer and an output layer
	- Nonlinearity was initially ReLU (we'll talk about this later), and my friend helpfully suggested Tanh as the final activation.
		- This lets me map position values to [-1,1] and amplify decisions that are close to 0
	- My compute is regrettably very limited, and the number of filters and size of dense layer is hyperparameter
5. Implement the trainer class. The optimization is done by pytorch and the RL method is the naive monte carlo one:
	- Play a game against an opponent. Suppose the states are $s_1 ... s_n$ and final result is $r$. Then we'll train the value function against the input pairs $(s_i, \gamma^{n-i}r)$ where $\gamma=0.9$ is chosen as the decay factor. Mean squared error is the metric I chose to optimize against
	- Exploratory behavior also had to be programmed in. I chose the simple $\epsilon$-greedy method, with $\epsilon$ scheduled some way
	- Initially, the opponent is fixed to be the OneMoveLookAhead() function. Later, this would be modified into a more advanced training pipeline similar to how alphaGo was trained.
	- I also intend to implement proper bootstrapping (SARSA style) at some point but I think that it is not the highest priority right now, I have enough choices to deal with at present


## Experiments and experiences messing with all sorts of cooked problems

Even after copius amounts of unit tests, it was still very difficult to get anything working. I'll detail my observations and some of the more cooked bugs here. This doesn't even cover the various bugs I met while implementing the enviroment (the creepiest was ``np.randomchoice`` casting the ``pyInt``s  in my list of valid moves into ``int32`` and that causing the bitboard to overflow when it was passed into the enviroment I implemented)

### Phase 1: Training against Fixed agent

The most pressing thing on the agenda was to get a neural net that wasn't awful. This is actually pretty hard to do. Some parameters that I needed to tune urgently included:
- The epsilon schedule
- The optimizer
- The network choice (arch1 or arch2)
- The size of the network 

First, some baseline parameters were used that usually are decent from literature
- epsilon scheduled to decay exponentially, initially from 1 to a minimum level of 0.01 after $80\%$ of input games have been run. 
- 10000 training games
- Adam optimizer with learning rate 0.001

And subject to this a $2\times 2$ grid search with 'small' and 'large' network sizes (large is twice the width of small) as well as both architectures were made. I would first try training against MCTS with 50 rollouts per move.

Initially, none of the methods demonstrated significant learning. After looking at neural network predictions, it was realised neural network was always outputting the same value irregardless of input, and this was as we were running into the classical 'dead neuron issue', where a neuron is rendered dead due to its linear weights being badly set. 

Researching solutions, I decided to switch relu to LeakyRelu (leaky = 0.01 gradient) and also decrease learning rate to 0.0001. Adam was left as is as it adapted the step sizes according to historic gradient which is quite nice I guess.

After rerunning the experiment in this setting, I still wasn't successful in getting any of the 4 networks to learn anything. I figured then it was because MCTS(50) was too good and it was only ever losing games (perhaps it is important in these situations to receive combination of positive and negative results. 

So I switched to training against OneMoveLookAhead, and finally I saw wisdom in the neural networks.  First, Arch2 failed miserably, and so did Arch1-small. They never managed to win more than 50% of time (in 100 games) against OneMoveLookAhead, but Arch1-big managed to win 86 in 100 games (randomise who goes first) which is very impressive. This corresponds to arc1-64-exponential.pth. I suppose the major lesson here is that neural net size is very important, some behavior you can only learn for sufficiently wide neural net (pleasingly, the learning was quick too, and the 10000 games were played in a few minutes, the bithacking had paid off!).

Lastly, I tested some naive bootstrapping but it didn't work very well (it seems tricky to use bootstrapping well).

Realising that neural net size mattered, I upped the size again to 100 conv filters & nodes in dense layer, and retrained, achieving similar results to the 64 node case (i.e Arch1-big). Then, I played against the AI and analysed its behavior. Here is what I observed
- The thing is actually pretty smart sometimes. It has figured out that playing in the middle is good,  knows to cluster its tokens together and also knows to complete 4 in a row
- But its also pretty dumb. It plays much worse as player 2 than player 1 (for some reason) but also lets the opponent complete 4 in a row pretty often (rather than blocking)

But overall, I'll call this a success. It was great to finally see signs of intelligence after a lot of work.

### Phase 2: Self-Playing

Now that we can beat OneMoveLookAhead() fairly reliably, it has come time to see how much better we can do. To this end, I first tried finetuning our previous agent against MCTS(50) out of curiosity, but this failed miserably and instead made the agent much worse (I'm not sure why this is).

So I decided to implement the self-play procedure, in style somewhat reminiscint of alphaGo: (I created a branch in the repo for this). For transparency Adam was switched to good old SGD.


```
CANDIDATE_OPPONENTS = [SOME BASELINES]
REPEAT N times:
	INITIALISE CURRENT AGENT FROM PREVIOUS AGENT PARAMS
	FOR M LEARNING ITERATIONS
		DECAY EPSILON
		USE NAIVE MONTE-CARLO LEARNING:
			PLAY CURRENT AGENT AGAINST RANDOM PREVIOUS CANDIDATE

	DECAY LEARNING RATE BY HALF
	ADD CURRENT AGENT TO OPPONENT LIST
```

I'm currently experimenting this, and trying to fix problems like exploding gradients etc (with L2 regularization etc). I'll update this if something cool happens.
