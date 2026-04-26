## Introduction 
When thinking about Artificial intelligence it is now common to think about neural networks and deep learning foremost. Deep learning has indeed prooven capable of solving complex task in a general way. However it is not the only way to inbue machines with intelligence, in fact for the longest of times it was not even the most popular and common method.

There are whole classes of problems out there for which deep learning is simply not a suitable solution. The general characteristic of this problems are: lack of structured data or patterns in the data, difficulty in establishing clear optimisable objectives, need for interpretable, require deterministic rule following, etc. Examples of such problems are role playing games, sequential control and planning.

Search is a powerful tool and a good match for such problems. The idea behind search is simple, a problem is represented as a state space and a solution to the problem is a path from a start state to a desired goal state. The first step of solving any problem through search is therefore task representation.

## Problem Representation
The goal is to represent our problem as a state space with multiple nodes. Each node represents a single state we can be in. To find solutions we move from one state to the next until we find a desired goal state. A solution to the problem is hence a sequence of all the states we went through from the start state till we reach the goal state, otherwise called a path.

Formally we define a problem as a 5 tupe $<S,A,T,s0,S_g>$ where,
S $\rightarrow $ the set of all states.
A $\rightarrow$ the set of actions we can take.
T(s,a) $\rightarrow$ is a transition function that tells what new state we are in after taking action a in state s. 
s0 $\rightarrow$ is the start state.
$S_g$ is the set of goal states.

### Example 1: Cleaning Robot
Suppose we have a cleaning Robot whose task is to clean the house. We have two rooms in the house and we want both of them clean. The robot can move to any room and can also clean. 

We can represent this problem as a simple state space, we can have a total of four states: both rooms are dirty, room 1 is clean but room 2 is dirty, room 2 is clean but room is dirty and both rooms are clean. Hence we can construct S to be:

$S = {CC, CD, DC, DD}$

The robot has three posible actions: go to room 1, go to room 2, and clean. We can then have A be:
$A = {Go to room 1, Go to room 2, Clean}$

let's assume initially both rooms are dirty hence, $s0 = DD$
Our goal is for both room to be clean so we have only one goal state: $S_g = {CC}$

The only thing left is to define the transition function:
$T(DD1, Go to room 1) \rightarrow DD1$
$T(DD1, Go to room 2) \rightarrow DD2$
$T(DD1, Clean) \rightarrow CD1$
$T(DD2, Go to room 1) \rightarrow DD1$
$T(DD2, Go to room 2) \rightarrow DD2$
$T(DD2, Clean) \rightarrow DC2$
$T(CD1, Go to room 2) \rightarrow CD2$
$T(CD2, Clean) \rightarrow CC2$



### #xample 2: Missionaries and Cannibal

## Uninformed Search 

Let's consider the Missionary and cannibal problem stated above. To find a solution to the problem we just need to traverse the tree and find a path that leads to the goal node from the start node. Thankfully we know of many tree traversal algorithms like Breath search first(BFS) and Depth search first(DFS).

```python
    
```

BFS is optimal and complete, meaning the solution it finds will always be the one that leads to the goal state in the least number of steps and that it is guaranteed to find the solution if one exists.

DFS on the other hand is not optimal since we could follow one branch k levels deep and find a solution while one solution exists at l levels deep l<k. It is also not complete because if the tree is not finite we can follow one branch indefinitely.
The advantage of DFS however is the reduced amount of memory required to hold the nodes while traversing the tree. It's space complexity is O(mb) compared to O(b^d) for BFS.
There are variations of DFS namely Depth limited Search and Iterative Deepening search which is complete and optimal while retaining the space complexity advantage making it the prefered choice for most problems in practice.

Now you may be asking how is any of this intelligent? We are just exhaustively searching the state space no? You are completely right. Nothing about the algorithms i described is intelligent. We will need to add something to them to make them be. That something is called heuristic functions

## Heuristic Functions
A heuristic function is a function that gives us information about the "goodness" of a certain state. They allow us to search the tree in a much more efficient manner knowing which branches are hopeless. They Essentially give us knowledge about the problem we are trying to solve.

More Formally, ley $\Pi$ be a problem with state space $S$. A heuristic function for $\Pi$ is a function $h:S\rightarrowR^+ U {\infinity}$ so that for every goal state S, we have h(s) = 0.

We want our heuristic h to be as informative and accurate as possible while having little overhead to compute. This is a tradeoff that needs to be considered carefully when we design such functions. Having the most accurate heuristic function will require solving the problem in the first place so we want a good enough approximation.

## Informed Search
Equiped with our heuristic functions now we can perform "Intelligent" search. One such algorithm is called A* Search.

```python

```

If we used A* for our path finding problem instead of UCS it would look like this:


A* is both complete and optimal.



## Local Search

## Evolutionary Search

## Adversarial Search

## Monte Carlo Tree Search

## Alpha Go

