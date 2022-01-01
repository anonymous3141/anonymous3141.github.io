---
excerpt: Me trying to do reinforcement learning

---

{% include head.html %}
# My First Attempt at Reinforcement Learning

In the current university holidays I decided to try my hands on Reinforcement learning, as the idea of an neural net (or other model) working out how to do something seems very cool. I'll go through what I did and what I learnt, although the analysis is severely limited by my lack of understanding of just what I am doing.

## OpenAI Gym Cartpole
After reading some Barto & Sutton and watching David Silver's excellent lecture series (well the first 8 lectures anyway), I felt confident enough (foolishly) to start implementing the algorithms. Gym's Cartpole enviroment was a nice one to dip my legs in and I implemented three different methods to solve it:
- Monte Carlo Tree Search
- Q learning on a small neural net with experience replay as per https://arxiv.org/pdf/1312.5602.pdf
- Policy Gradient with a value function baseline, with the value function as neural net learnt dynamically 

Ultimately, all 3 methods successfully solved the problem. You can find the code I wrote [here](https://github.com/anonymous3141/Connect4/tree/main/cartpole)

### Monte Carlo Tree Search
This method worked very well, even with a purely random rollout policy and basic epsilon greedy tree policy (UCT with unvisited children prioritised worked well too) quickly got to the top performance of 500 reward. I am very glad I started lazy and just coded the very basics. But this requires a precise model to simulate which doesn't work well in real life.

### Q Learning
Of the actual "model-free" methods Q-learning performed best. The state-action function was approximated by a small 4 layer neural network, optimized by Adam with a standard learning rate. Parameters like $\epsilon$ was kept to "sensible" values. 

I used experience replay and a previous cached model to provide target $q_{max}$ to increase the stability of the Q values as otherwise Q values might blow up (this is not theoretical actually, I had this happen with Connect4, see below)

Within around 1000 episodes the agent was consistently able to obtain the maximum score. It was pleasing that I didn't have to tune any hyperparameters to achieve this.

### Policy Gradient via REINFORCE algorithm
This is probably the method I understood the least as it is quite fiddly. The main idea I suppose is to have a model outputting a stochastic policy for the given state, and use gradient ascent to ensure state-action pairs that yield good returns on average have their probabilities increased. A value function is also introduced as a "baseline" to reduce variance and thus increase learning speed. While you have the peace of mind of not having to play with Epsilon greedy (exploration is handled by the innate stochasticity of the policy I think) you do now have two networks to sort out now.

I implemented the REINFORCE algorithm with a value baseline two different ways. One used a multiheaded neural network that both outputted a policy as well as a state value. The other used distinct value and policy networks. In either case, the policy network was optimised the standard policy gradient way and the value network trained by Monte Carlo Learning which sufficed here despite being noted for high variance.

The multiheaded network trained quite fast but was very fiddly. It turned out that if you used the same learning rate on optimising the two heads it didn't work, and you had to choose a higher LR for policy than value (maybe I did something wrong). Fortunately the two network way was easier, but either way after 1500 episodes I was getting consistent 500 scores.

### Comments on Gym
Gym is a really nice enviroment with reasonably well documented OO interface for each enviroment. A trick that they didn't mention was deepcopying the enviroments for simulation (as required in MCTS). To see full interface for each enviroment the function ``dir(env)`` does the job.

## Connect 4

So my ultimate goal was to make an connect 4 agent on the standard $6\times 7$ board in the style of alphaGo, developing a position evaluation function and strapping it to an MCTS and hopefully see it play decently. Turns out this is much easier said than done. Compared to Cartpole, the problem pose several obvious increases in difficulty

### Challenges

1) Cartpole had a two dimensional state. Connect4, encoded in the sensible way as 2 42 bit bitmasks, is 84-dimensional which means optimisation is harder
2) The state space is much sharper: Small differences in state create big differences in position value. This makes learning harder and slower
3) Gradients are sparser. Reward signals come once per episode which also makes learning harder
4) For self-play training, we need to make an agent that can play both as first and second players.

### My Approach
Our job consists of 3 parts (aside from writing correct code which is actually quite hard to do because you can't really tell if the agent is training incorrectly)

**Part 1:** Implement a fast enviroment.

This was relatively simple and I followed Pascal Pons' [bitmask implementation](http://blog.gamesolver.org/solving-connect-four/06-bitboard/) which is very careful. I however had to write unit tests as the correctness of the enviroment was imperative. I effectively implemented, with a few extensions, the interface of Gym enviroments

**Part 2:** Train a neural agent.

More on this below

**Part 3:** Strap said agent to a MCTS or a depth constrained minimax algorithm.

More on this below, but games like Connect 4 simply needs some lookahead to do well.

### Key Points in implemention
***Invalid Move masking***: In all circumstances neural network output for invalid moves were ignored. This is trivially doable in DQN and in policy gradient we simply softmax over outputs that are legal moves.

***Neural network library:*** I use Pytorch running on my laptop CPU. Not the best compute but it'll do I suppose

***Neural architecture:*** I used a very small CovNet in all approaches with ~5000 trainable parameters, containing 2 convolutional layers and 1 dense hidden layer as my neural architecture. The input was a $2\times 6\times 7$ boolean tensor describing the red and yellow tiles PLUS a boolean value indicating whose turn it is. The boolean value was fed in together after the convolutions were done. The output layer was a single neuron in value network, anbd 7 neurons in policy and Q networks (for obvious reasons)
### Me trying to write code
The code for part 2 was quite hard, and often required me to use pytorch in nonstandard and hacky ways. Also there was a lot of parts of the algorithm, especially with respect to say state transition. I can't guarantee my code is correct QaQ.

The codebase was designed to be as modular as possible with a file containing models and model evaluation functions, a file containing the enviroment, a file containing tests and training files. A further program allows user to play against selected models. Neural net parameters were periodically saved to designated output files. You can find the codebase [here](https://github.com/anonymous3141/Connect4/tree/main/RLConnect)
### Part 2a (training.py)
I took inspiration from AlphaGo's training procedure.

On a very high level (and I hope I got this right I'm not sure), AlphaGo uses supervised learning to train a policy network, then improves it by REINFORCE (no baseline) via playing against past versions of the policy network $\pi$. After that, a value network $v_\theta$ is trained to approximate the value function of $\pi$, i.e  $v_{\theta}(s)\approx v_{\pi}(s)$, before using $v_{\theta}$ as a baseline to REINFORCE to make another training pass. This, combined smartly, produces the tree policy and leaf value estimator that along with a smaller supervise-trained rollout policy network form the basis of the MCTS algorithm that achieved superhuman performance. 

I implemented the REINFORCE algorithm as with alphaGo, but with random initialization (I didn't have a copora of games to begin with). I tried Monte Carlo learning for $v_{\theta}$ but it just didn't work, likely due to reasons mentioned before (variance too high!) so I switched to $TD(0)$ instead. After training overnight playing 100K games the agent was still pretty garbage was showed some signs of intelligence, playing in the centre and about a third of the time blocking my lines. It even beat me once because I was being stupid...

### Part 2b (Qtraining.py)
I realised that the advantages of using policy gradient, namely the possibility of allowing supervised pretraining, didn't really apply to me here. So I switched to the method I was most comfortable with understanding, Q-learning. My first training attempt, leaving agent to train overnight, resulted in the gradients blowing up catastrophically. Lowering the learning rates and adding a small weight decay regularization fixed the issue.

Training for $10^5$ games using both experience replay and a cached target model yields a player that is still pretty stupid: It frequently misses 3 in a row and immediate wins. But it does play reasonably intelligent moves sometimes (it can block 3 in a row lines about 50% of the time and has figured out to play in the centre)

### Part 3
The Q Network, despite being pretty bad, was used in as an evaluation in a 5 move (i.e 3 of your own, 2 of the opponent) lookahead negamax. Along with a transposition table (position cache) a move takes about 3 seconds to calculate. This produced an agent that was not great, but played at a "non-stupid" level.

### Future Work
Admitted the training time given was quite short (looking at what other people have done, they seemed to have trained for many days), and the results were a bit dissappointing. 

I suspect that my neural network might have been ill designed but in interests of time I avoided tinkering with designs and/or hyperparameters.

It is somewhat regrettable that I didn't find the time to try implementing AlphaGo Zero's algorithm which strapped in MCTS "as a powerful policy improvement operator". It is very interesting and promising here but I think I'd be overextending myself.

## Conclusion
So yeah, that's a good portion of  two weeks of my holiday right there. Time well spent, I suppose. You can find all code [here](https://github.com/anonymous3141/Connect4/tree/main/RLConnect). I might get my friends in on this some time, see what they can do to make the agent less dumb.
