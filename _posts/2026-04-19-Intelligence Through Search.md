## Introduction 

## Problem Representation

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

