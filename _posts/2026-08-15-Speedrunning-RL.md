
In his 1954 paper Alan turing conjectured that acheive machines that can think it might be easier to prgram a child's brain than an adults brain.
This is the whole philosophy of reinforcement learning, we want to program a child's brain that can learn by interactions with it's environment just like humans and animals do.
Reinforcement learning refers to both the problem of learning through interactions and the set of techniques and algorithms that employ an interaction feedback loop to learn optimal behaviours. We may apply other types of solutions to reinforcement learning problems and may apply reinforcement learning solutions to other types of problems. I like to think of reinfocement learning as refering to reinforcement learning problems.

## Defining Terms
Let's start off by defining important terms in RL.

**Agent**: This is the entity that is interacting with the environment and learning. Any entity that can change the state of the environment by it's actions is an agent. There may be 1 or more agents.
**Policy**: $\pi(a|s) is a function that maps states to a probability distribution over actions. It determines the behavior of the agent by assigning probabilities to actions for given states. Such a policy that assigns probabilities to actions is called a stochastic policy. The policy may also be deterministic, it just maps each state to an action with probability 1.
**state**: this refers to the current configuration of the environment(environment state) which changes as the agent takes actions. State can also refer to the agent state in cases where the envrionment is not fully observable. For example robots have limited filed of vision, the environment they are in is only partially observable. In these cases the agent state is a subset of the environment state that the agent can observe.
**reward**: a scalar value the agent receives after every action it takes. It serves as a feedback signal quantifying how good that action was for the current state. 
**return
**model**: a model refers to the dynamics of the environment. It consist of the transition model $p(s' | s,a)$ and the reward model $p(r | s,a,s')$. Together the model is represented as p(s',r | s,a). RL solution can be either model-based(we learn or use a model) or model-free (we make no use of the model). 
**value functions**: this is a function that quantifies how "good" a state is. It estimates the expected sum of rewards starting from that state. By learning the value of states we can derive a policy by seeking high value states. There are two types of value functions State value functions V(s) and state-action value functions Q(s,a).


## Exploitation vs Exploration
Suppose you have a slot machine with two levers A, and B. you pull lever A and get 0$. You then pull lever B and get 10$. Which lever do you pull next? A or B? From you first two experiences pulling lever B should be better since lever A gave you 0$ the first time right? But what if lever A has 0.8 probability of giving 10$ and lever B 0.5? In that case lever A is the better choice. 

In the scenario above pulling lever B is known as exploitation. We are using the knowledge we have so far of the environment and go for the better option B. pulling lever A is called exploration. we are not sure if A is definitely bad so we try it out to see.

When to exploit and when to explore is at the core of every RL algorithm and is formally called the exploitation exploration tradeoff. In the scenario above suppose we do greedy action selection and always exploit our current knowledge. If on average our rewards are positive we will choose B every single time and will never know how good A actually is. if A is actually better than B like in the scenario above out regret is linear with the number of actions we take.  

One alternative to greedy action selection is $\epsilong$-greedy action selection. The idea is to set a probability $\epsilon$ with which we explore and with $1-\epsilon$ probability we exploit. This is better because in the limit we will converge to selecting the better action. The here is the problem, suppose we make the decision 100 gazillion times and a certain A is the better choice, we will still be choosing action B with $\epsilon$ probability even though it is not needed anymore. Our regret is still linear.

Another alternative for which regret is not linear but logarithmic is the upper confidence bound action selection(UCB). In UCB action selection depends on our current estimate of how good the action is and our degree of uncertainty of that action. Action with high rewards or high uncertainty are prefered. In the limit we converge to the actions with high rewards only. 

## Bellman Equation
We defined the value of a state as the expected return starting from that state:

$V(s) = E[G_t | s_t = s]$
$V(s) = E[R_t+1 + \gamma G_{t+1} | s_t = s]$
$V(s) = \sum_a \pi(a|s) \sum_s' \sum_r p(s', r | s,a)[r + \gamma E[G_{t+1} | S_t+1 = s']]$
$V(s) = \sum_a \pi(a|s) \sum_s' \sum_r p(s', r | s,a)[r + \gamma v(s')]$

This final equation is called the bellman equation. It defines the value of a state recursively in terms of the value of another state. This is important because in this form we can implement iterative algorithms that start with arbitrarily initialised values and converge to correct estimates. Remember that value learning is one of the common ways to learn optimal policies. Many of the RL algorithms use the bellman equation as an update rule for learning values and  use those value estimates to derive optimal policies.


## Prediction, Control and Dynamic Programming 
The RL problem can be subdivided into two main subproblems, the prediction problem and the control problem. Given a policy $\pi$ the task of evaluating how good that policy is is refered to as the prediction problem and the task of finding an optimal policy is called the control problem. Give a model of the environment p(s', r | s,a) we can use dynamic programming methods to solve the prediction and control problem. 

Let's first consider the simpler of the two, the prediction problem. We have an arbitrary policy $\pi$ and we want to know how good of a policy it is. We do so by learning the values of the states under that policy. The dynamic programming algorithm to do this is called policy evaluation. Since we have a model of the environment we can directly solve for value of each state using equation <bellman equantion ref here>. We initialise V(s) for all s \in S to 0 and for each ever s we do the computation in <bellman ref> until we reach convergence(we converge when the values do not change in successive iterations anymore). You can find a full implementation of the algorithm [here]().

Now let's consider the control problem. Once we know the values of a state under an arbitrary policy from the policy evaluation algorithm we can simply update the policy by greedily picking the action that maximises the expected return for each state. In other words the action that maximises $\sum_s' \sum_r p(s', r | s,a)[r + \gamma v(s')$. This algorithm is called policy improvement. You can find an implementation here.

Given an RL problem we can then find the optimal policy by alternating between policy evaluation and policy improvement. We start with an arbitrary policy $\pi$ for example the uniform random policy, perfor mpolicy evaluation then plicy improvement to get a new policy and evaluate that new policy and again perform policy improvement and so on until we converge to the optimal policy. This algorithm is called policy iteration and forms the basis of most of the RL algorithms. 

Policy iteration allows us to converge to optimal policies but there is a problem with it. It requires multiple policy evaluation steps which itself performs multiple sweeps throught the state space making the algorithm inefficient. A better algorithm is called value iteration where we directly update the policy for each state after a single evaluation step. This leads to much faster convergence. You can find a value iteration implementation here []().


## Monte Carlo prediction and Control
Dynamic programming algorithms can fing optimal policies but there are two major limitations with it. One it requires a model of the environment, without one we cannot directly solve for v(s) using the bellman equation. Second it requires very high computaional resources.

Fron here on we will consider methods where we learn from our agent's own interactions with the environment. First let's again consider the prediction problem but this time we have no model of the environment. How could we get estimates for the values then? Well we can start by taking actions following an arbitrary policy and see what rewards we get. We can then sample and entire episode and compute the return G_t by summing all the individual rewards we got throughout the episode. This return is not an estimate but the actual return from following our policy, we can use it to update the values of the states. For each state we consider only the return after the first time the state was visited and use that to update the value of that state. This algorithm is called monte carlo prediction. An implementation can be found here.

For control we can follow the same idea of policy iteration, after each prediction we update our policy greedily. This is called monte carlo control.

To make all this concrete let us consider the example of an agent learning to move in a maze. The agent can move left, right, up and down and each action give a reward of -1. The episode terminates when the agent reaches the goal. let's assume \gamma = 0.9.

We start by initiasing a uniform random policy where the agent is equally likely to get in either direction. Following this policy suppose the agent performs the sequence of actions (L,D,U,D,R,R,U,R). This is a single episode we sampled we can now compute the return


## Temporal Difference Prediction and Control
In monte carlo methods we sampled an entire episode then computed the return and used it to make our updates. But what if our episodes never end or are very long? Then throughout an episode we never learn anything. In TD methods everytime we take an action we observe the reward and use it to update our Q values. The simples TD learning algorithm is called SARSA. How it works is we take an action following the policy and observe a reward, we then consider the next action acording to the policy 

Another TD learning algorithm is Q learning where instead of considering the next action according to our policy we consider the greedy action. 

## Value learning with Neural Networks

## Policy gradients

