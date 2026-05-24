
## Introduction

When thinking about Artificial intelligence we usually think about neural networks and deep learning. Deep learning has indeed proven capable of solving complex tasks in a general way. However it is not the only way to imbue machines with intelligence, in fact for the longest of times it was not even the most popular method.

There are whole classes of problems for which deep learning does not offer a suitable solution. These are problems for which there is often a lack of structured data or the data has no learnable patterns, problems for which it is difficult to establish clear optimisable objectives, problems that require interpretable solutions etc. Examples of such problems are role playing games, sequential control and planning in for example robots.

Search is a powerful tool and a good match for such problems. The idea behind search is simple, a problem is represented as a state space and a solution to the problem is a path from a start state to a desired goal state. The first step in solving any problem through search is therefore properly representing the task as a state space.

## Problem Representation

The goal is to represent our problem as a state space with multiple nodes. Think of a tree in which each node represents a state we can be in. For any given problem we have to find what are the discrete states we can be in.

**Example 1: Cleaning Robot**

Suppose we have a cleaning Robot whose task is to clean the house. We have two rooms in the house and we want both of them to be clean. The robot can move to any room and clean it. What is the set of states the robot can be in?

Each room can be either clean(C) or dirty(D). We have two rooms so $2^2$ possibilities (both are clean, Room 1 is clean and Room 2 is dirty,...). Additionally our robot can be in either room at a time so that makes the total $2 \times 4 = 8$ possible states.

We can represent a state with the notation `<Room1 state><Room2 state><Robot position>`. The set of all possible states is then:

$$S = \{CC1, CD1, DC1, DD1, CC2, CD2, DC2, DD2\}$$

Now that we have a set of states we need to define how to transition between them. Transitions are triggered by actions and result in a new state. Our robot can take three actions: Go left to room 1, Go right to room 2 and clean.

$$A = \{L, R, C\}$$

In any state the robot can do any of those actions and will result in a new state. For example if current state is DD1 and robot takes action C the new state is CD1. On the other hand if the action R was taken the new state would be DD2.

Formally we define a problem as a 5-tuple $\langle S, A, T, s_0, S_g \rangle$ where:

- $S \rightarrow$ the finite set of states.
- $A \rightarrow$ the finite set of actions.
- $T(s,a) \rightarrow$ is a transition function that tells what new state we are in after taking action $a$ in state $s$.
- $s_0 \rightarrow$ is the start state.
- $S_g \rightarrow$ is the set of goal states.

Suppose initially both rooms are dirty and the robot is in the left room, $s_0 = DD1$. Our goal is for both rooms to be clean so our goal states are: $S_g = \{CC1, CC2\}$.

Finding a solution to this problem is now finding a sequence of actions (a path) that lead from the start state to the goal state. To do that we need search algorithms.

## Uninformed Search

### Breadth First Search

BFS is a simple search algorithm that traverses the nodes in a graph layer by layer until a termination condition is met or the full tree is traversed.

```pseudocode
function BFS(problem):
    frontier = Queue()
    frontier.enqueue([problem.start_state])
    visited = {}

    while frontier is not empty:
        path = frontier.dequeue()
        current_state = path.last()

        if problem.is_goal(current_state):
            return path

        if current_state not in visited:
            visited.add(current_state)
            for each action in problem.actions(current_state):
                next_state = problem.transition(current_state, action)
                if next_state not in visited:
                    frontier.enqueue(path + [next_state])

    return failure
```

BFS is optimal, the solution it finds will always be the one that leads to the goal state in the least number of steps(since we traverse the tree layer by layer we will visit all the nodes at level k before visiting nodes at level k+1). It is also complete, finding the solution is guaranteed if one exists.

### Depth First Search

DFS is a similar search algorithm except it traverses the tree depth wise. That is for each branch it traverses the full branch no matter how many levels deep before backtracking.

```pseudocode
function DFS(problem):
    frontier = Stack()
    frontier.push([problem.start_state])
    visited = {}

    while frontier is not empty:
        path = frontier.pop()
        current_state = path.last()

        if problem.is_goal(current_state):
            return path

        if current_state not in visited:
            visited.add(current_state)
            for each action in problem.actions(current_state):
                next_state = problem.transition(current_state, action)
                if next_state not in visited:
                    frontier.push(path + [next_state])

    return failure
```

DFS is not optimal since we could follow one branch $k$ levels deep and find a solution while a shorter solution exists at $l < k$ levels deep in another branch. It is also not complete because if the tree is not finite we can follow one branch indefinitely.

The advantage of DFS however is the reduced amount of memory required to hold the nodes while traversing the tree. Its space complexity is $O(mb)$ compared to $O(b^d)$ for BFS where m is the maximum depth, b is the branching fator and d is the depth. 

There is a variation of DFS called Iterative Deepening Search which is complete and optimal but retains the space complexity advantage making it the preferred choice for most problems in practice.

Let's look at an example of using these algorithms to solve a problem.

**Example 2: Missionaries And Cannibals Problem**

**Problem Statement**

Three missionaries and three cannibals need to cross a river. There is one boat that can carry at most two people. If cannibals ever outnumber missionaries on either bank, the missionaries will be eaten. Find a sequence of crossings that gets everyone safely to the other side.

**State Space**

We represent a state as a triple $(M, C, B)$ where $M$ is the number of missionaries on the starting bank, $C$ is the number of cannibals on the starting bank, and $B \in \{1, 0\}$ indicates whether the boat is on the starting bank (1) or the destination bank (0).

$$s_0 = (3, 3, 1) \quad \text{— everyone starts on the left bank.}$$

$$S_g = \{(0, 0, 0)\} \quad \text{— everyone has crossed to the right bank.}$$

A state is valid if missionaries are never outnumbered on either bank (unless there are zero missionaries on that bank).

!()[../assets/search/mc_state_space.png]

**Finding a Solution with BFS**

BFS explores states level by level, guaranteeing the shortest solution. Starting from $(3,3,1)$, it enqueues all reachable states from one crossing, checks validity, then expands those states, and so on. Below is the sequence BFS discovers:

```
(3,3,1) → send 2 cannibals                → (3,1,0)
(3,1,0) → return 1 cannibal               → (3,2,1)
(3,2,1) → send 2 cannibals                → (3,0,0)
(3,0,0) → return 1 cannibal               → (3,1,1)
(3,1,1) → send 2 missionaries             → (1,1,0)
(1,1,0) → return 1 missionary, 1 cannibal → (2,2,1)
(2,2,1) → send 2 missionaries             → (0,2,0)
(0,2,0) → return 1 cannibal               → (0,3,1)
(0,3,1) → send 2 cannibals                → (0,1,0)
(0,1,0) → return 1 cannibal               → (0,2,1)
(0,2,1) → send 2 cannibals                → (0,0,0) ✓
```

BFS finds this 11-step solution and guarantees no shorter solution exists.

**Finding a Solution with DFS**

DFS dives immediately down one branch of the state tree. It may explore many dead ends and invalid states before backtracking to a valid path. It will eventually find a solution (given a finite state space with cycle detection) but it may not be the shortest one. For instance, DFS might explore the branch where one missionary and one cannibal cross first, hit a dead end after several moves, backtrack, and eventually converge on a valid sequence — potentially longer than 11 steps. Without iterative deepening, there is no guarantee on solution length.

---

Now you may be wondering how any of this is considered intelligence. We are just exhaustively searching the state space until we find what we need. Surely this is not the smartest way to solve tough problems. Search can be very powerful, it can beat chess and Go champions but to get there we need an extra ingredient in our search algorithms. That ingredient is called heuristic functions.

## Heuristic Functions

A heuristic function is a function that gives us information about the "goodness" of a certain state. They allow us to search the tree in a much more efficient manner knowing which branches are promising and which are hopeless. They essentially give us knowledge about the problem we are trying to solve.

More formally, let $\Pi$ be a problem with state space $S$. A heuristic function for $\Pi$ is a function $h: S \rightarrow \mathbb{R}^+ \cup \{\infty\}$ so that for every goal state $s$, we have $h(s) = 0$.

We want our heuristic $h$ to be as informative and accurate as possible while having little overhead to compute. This is a tradeoff that needs to be considered carefully when we design such functions. Having the most accurate heuristic function will require solving the problem in the first place so we want a good enough approximation.

## Informed Search

### A* Search

The idea behind A* is simple, for each state we compute the estimated cost (using our heuristic function) to reaching the goal and the cost incurred so far. Adding both of them gives us the cost of a state. We greedily choose the action that leads to the state with lowest cost.

```pseudocode
function A_STAR(problem, h):
    start = problem.start_state
    frontier = PriorityQueue()
    frontier.enqueue(path=[start], priority=h(start))
    visited = {}

    while frontier is not empty:
        path = frontier.dequeue_min()
        current_state = path.last()

        if problem.is_goal(current_state):
            return path

        if current_state not in visited:
            visited.add(current_state)
            for each action in problem.actions(current_state):
                next_state = problem.transition(current_state, action)
                if next_state not in visited:
                    g = path.cost()          // cost so far
                    f = g + h(next_state)    // estimated total cost
                    frontier.enqueue(path + [next_state], priority=f)

    return failure
```

A* is both complete and optimal, provided the heuristic $h$ is admissible — meaning it never overestimates the true cost to reach the goal.

Let's consider an example of using A*.

**Example 3: Robot Pathfinding on a Grid**

**Problem Statement**

A robot is placed on a 5×5 grid and must navigate from the top-left corner $(0,0)$ to the bottom-right corner $(4,4)$. Several cells are blocked by obstacles. The robot can move up, down, left, or right one cell at a time. Each move has a cost of 1. We want the shortest unobstructed path.

The grid looks like this (S = start, G = goal, X = obstacle):

```
S  .  .  X  .
.  X  .  X  .
.  X  .  .  .
.  .  .  X  .
X  .  .  .  G
```

**Finding the Solution using A\***

We use the Manhattan distance as our heuristic:

$$h(x, y) = |4 - x| + |4 - y|$$

This is admissible because on an open grid with unit costs, Manhattan distance is the true minimum number of moves, and obstacles can only increase the actual cost, never decrease it.

A* maintains a priority queue ordered by $f = g + h$. Starting at $(0,0)$ with $g=0$ and $h=8$, it expands the most promising state at each step. States near obstacles are deprioritized because their detours increase $g$ faster than $h$ decreases. The algorithm discovers the following optimal path of length 8:

```
(0,0) → (1,0) → (2,0) → (2,1) → (2,2) → (2,3) → (2,4) → (3,4) → (4,4)
```

A* finds this path in far fewer node expansions than BFS would, because the heuristic steers it south and east rather than exploring fruitless detours. BFS would have expanded every cell at distance 1, then distance 2, and so on — evaluating many states that A* never even considers.

## Local Search

So far we have seen how to exhaustively search the state space and how to use heuristics to optimally do so. There are still some types of problems however whose state space is ridiculously large and searching through them taking one action at a time will take tremendous time. Additionally there are types of problems who typically do not have one exact solution but multiple that are "good enough". An example of such a problem is the N-Queens problem.

**Example 4: N-Queens Problem**

**Problem Statement**

Place $N$ queens on an $N \times N$ chessboard such that no two queens threaten each other — that is, no two queens share the same row, column, or diagonal. For $N = 8$ this is the classic 8-Queens problem.

**State Space Size**

A naive representation places one queen in each of the $N^2$ cells, giving $\binom{N^2}{N}$ possible configurations — for $N = 8$ that is over 4 billion states. Even with the constraint that exactly one queen appears per row (reducing the space to $N^N$), we still have $8^8 = 16{,}777{,}216$ states for $N = 8$. Exhaustively searching this space is expensive, and it only grows worse as $N$ increases.

**Multiple Solutions Exist**

The 8-Queens problem has 92 distinct solutions (12 if reflections and rotations are excluded). We do not need to find all of them — any single valid arrangement is acceptable. This makes it a natural fit for local search: rather than finding the one optimal path, we simply need to land in any valid state.

Local search is a great tool to throw at these types of problems. The idea of local search is simple: we start with a randomly chosen initial state and look for its neighbouring states. These are states that can be reached in $K$ actions, and we say $K$ is the size of the neighbourhood. From the neighbouring states we pick one with a better cost than the previous state.

```pseudocode
function LOCAL_SEARCH(problem, max_iterations):
    current = problem.random_initial_state()

    for i = 1 to max_iterations:
        if problem.is_goal(current):
            return current

        neighbours = problem.get_neighbours(current)
        best_neighbour = argmin_{s in neighbours} cost(s)

        if cost(best_neighbour) >= cost(current):
            return failure   // stuck at local minimum

        current = best_neighbour

    return failure
```

This algorithm can be interpreted as hill climbing in the cost landscape — at each step we move to the neighbouring state with the lowest cost.

**Solving the 8-Queens Problem with Local Search**

Represent a state as an array $[q_1, q_2, \ldots, q_8]$ where $q_i$ is the row of the queen in column $i$. The cost function is the number of attacking pairs of queens (we want this to reach 0). Neighbours are all states reachable by moving a single queen to a different row within its column, giving $8 \times 7 = 56$ neighbours per state.

Suppose we start with the random initialisation $[2, 4, 6, 8, 3, 1, 7, 5]$ (1-indexed rows):

- **Step 0:** Cost = 5 attacking pairs.
- **Step 1:** Move queen in column 3 from row 6 to row 5. Cost = 3.
- **Step 2:** Move queen in column 6 from row 1 to row 3. Cost = 1.
- **Step 3:** Move queen in column 8 from row 5 to row 7. Cost = 0.

In this case local search finds a solution in just 3 steps. By contrast, A* on the full state space would need to expand thousands of nodes to guarantee finding a goal state. For $N = 8$, local search finds a solution in roughly 3–5 steps on average when starting from a favourable neighbourhood — a dramatic reduction compared to exhaustive search.

From this example we can also see some apparent limitations of this approach:

1. **Dependence on the initial state.** If we start in a bad neighbourhood we may never find a solution no matter how long we search. For example, starting with $[1, 1, 1, 1, 1, 1, 1, 1]$ (all queens in row 1) places us in a region where every neighbour still has high conflict and the algorithm quickly gets trapped. To solve this problem we can **restart the search** with a new random initial state whenever we are stuck. On average with random restarts it takes about 4 attempts to find a solution, but our success rate jumps to nearly 100%.

2. **Local minima and plateaus.** We can get stuck in a local minimum or a plateau where no neighbouring state is better than the current state but the current state is not a solution either. For example, the state $[1, 5, 3, 6, 4, 2, 8, 7]$ has a cost of 1 — one attacking pair — yet every single neighbour also has cost $\geq 1$, so the algorithm halts without having found a solution. To solve this problem we can allow **sideways moves**: with some probability $p$ we permit moving to a neighbour with equal cost rather than strictly lower cost, allowing us to traverse plateaus and escape shallow traps.

A variant of local search is **local beam search**, where we maintain $N$ random initial states simultaneously and search all of their neighbourhoods at each step. Rather than keeping all neighbours independently, we select the $N$ best-scoring states from the combined pool of neighbours and continue from those. This provides the benefit of parallelism and diversity, reducing the chance that all threads of the search get stuck simultaneously.

## Genetic Algorithms

Genetic Algorithms (GAs) are a class of search-based optimisation techniques inspired by evolutionary biology. There are hard limitations we cannot deal with using local search like problems with jagged, mountainous cost landscapes with a high number of local minima, and problems with a deceptive peaks where increasing fitness locally leads away from the global optimum. GAs handle these settings more robustly by maintaining a diverse population of candidate solutions and combining them.

GAs use a lot of biological terminology. A **population** is the set of candidate solutions being considered (good or bad solutions alike). A **chromosome** is one individual solution sampled from the population. A **gene** is one element within a chromosome (e.g. the row position of one specific queen). An **allele** is the value a gene takes (e.g. row 4).

Genetic algorithms involve three stages: **Selection**, **Crossover**, and **Mutation**. To understand how it works let's consider an example.

**Example 5: 8-Queens with a Genetic Algorithm**

We represent each chromosome as an array of 8 integers, where position $i$ holds the row of the queen in column $i$. For example:

```
Chromosome A: [2, 4, 7, 3, 6, 8, 5, 1]
Chromosome B: [3, 6, 2, 7, 1, 4, 8, 5]
Chromosome C: [5, 1, 8, 4, 2, 7, 3, 6]
Chromosome D: [1, 7, 4, 6, 8, 2, 5, 3]
```

The fitness of each chromosome is the number of **non-attacking pairs** of queens. The maximum is $\binom{8}{2} = 28$, which corresponds to a valid solution. Higher fitness is better.

```
Fitness(A) = 24    Fitness(B) = 23    Fitness(C) = 22    Fitness(D) = 20
```

**Selection**

In selection we sample fit chromosomes from the population to serve as parents. We can select **by rank** (choosing the top $K$ chromosomes) or by **fitness-proportionate selection** (roulette wheel), where the probability of choosing a chromosome is proportional to its fitness:

$$P(\text{select } X) = \frac{\text{fitness}(X)}{\sum_i \text{fitness}(i)}$$

In our example the probabilities are approximately $A: 0.27$, $B: 0.26$, $C: 0.25$, $D: 0.22$. We sample two pairs of parents: $(A, B)$ and $(C, D)$.

**Crossover**

In crossover we combine two parent chromosomes to produce offspring. In **single-point crossover** we pick a random cut point $k$ and swap the tails:

```
Parent A: [2, 4, 7 | 3, 6, 8, 5, 1]    cut at k=3
Parent B: [3, 6, 2 | 7, 1, 4, 8, 5]

Child 1:  [2, 4, 7, 7, 1, 4, 8, 5]
Child 2:  [3, 6, 2, 3, 6, 8, 5, 1]
```

In **multi-point crossover** we use multiple cut points, alternating segments from each parent:

```
Parent C: [5, 1 | 8, 4, 2 | 7, 3, 6]    cuts at k=2, k=5
Parent D: [1, 7 | 4, 6, 8 | 2, 5, 3]

Child 3:  [5, 1, 4, 6, 8, 7, 3, 6]
Child 4:  [1, 7, 8, 4, 2, 2, 5, 3]
```

**Mutation**

To maintain diversity and avoid premature convergence, we randomly alter alleles in the offspring with a small probability $p_m$ (typically 1–5%). Common mutation operators include:

- **Swap mutation:** randomly pick two genes and swap their values.
  `[2, 4, 7, 7, 1, 4, 8, 5]` → swap positions 1 and 4 → `[2, 1, 7, 7, 4, 4, 8, 5]`
- **Random resetting:** replace a gene's allele with a randomly chosen new value.
  `[3, 6, 2, 3, 6, 8, 5, 1]` → reset position 3 → `[3, 6, 2, 5, 6, 8, 5, 1]`
- **Scramble mutation:** randomly shuffle a subset of genes.
  `[5, 1, 4, 6, 8, 7, 3, 6]` → scramble positions 3–5 → `[5, 1, 4, 7, 6, 8, 3, 6]`

After selection, crossover, and mutation, the new offspring replace the least-fit members of the population and the cycle repeats. Over generations the population converges toward high-fitness chromosomes. GAs typically find a valid 8-Queens solution in fewer than 100 generations, and they scale far more gracefully than exhaustive search as $N$ grows.

## Adversarial Search

All the search methods discussed so far are only applicable if there is a single agent in the search tree. There could be two or more agents in a search tree, each looking for a solution that meets its own goal criteria. A good example is board games like Go or Chess — two players, both trying to win, with directly conflicting goals. This is called **adversarial search**.

In adversarial search, when traversing the tree we need to consider the intention of the other agent and take paths that maximise our benefit regardless of what the other agent does. One algorithm that does this is the **minimax algorithm**.

**Minimax Algorithm**

We label our agent MAX (trying to maximise the outcome) and the opponent MIN (trying to minimise it). At each node in the tree, if it is MAX's turn we pick the child with the highest value; if it is MIN's turn we pick the child with the lowest value. The intuition is that on our turn we should maximise our outcome and on the opponenents turn we assume he would take the path that minimises our outcome. Leaf nodes are evaluated with a terminal utility function.

```pseudocode
function MINIMAX(state, is_max_turn):
    if state is terminal:
        return utility(state)

    if is_max_turn:
        best = -∞
        for each successor s of state:
            best = max(best, MINIMAX(s, false))
        return best
    else:
        best = +∞
        for each successor s of state:
            best = min(best, MINIMAX(s, true))
        return best
```

**Example 6: Tic-Tac-Toe with Minimax**

Consider a late-game Tic-Tac-Toe position where it is X's (MAX) turn:

```
X | O | X
---------
O | X | O
---------
  |   |  
```

X can play in positions $(3,1)$, $(3,2)$, or $(3,3)$. Minimax expands the subtree rooted at each move and assigns terminal values: +1 for X wins, -1 for O wins, 0 for draw.

```
X plays (3,1):
    O plays (3,2): board full → Draw (0)
    O plays (3,3): board full → Draw (0)
    MIN value: 0

X plays (3,2):
    O plays (3,1): board full → Draw (0)
    O plays (3,3): board full → Draw (0)
    MIN value: 0

X plays (3,3):
    X wins immediately (diagonal X-X-X) → +1
    MIN value: +1
```

MAX chooses $(3,3)$ with value +1 — the winning move. Minimax guarantees that X will win (or at worst draw) against any opponent, because it assumes the opponent always plays optimally.

**Alpha-Beta Pruning**

There is an important improvement we can make to minimax. If we track a range $[\alpha, \beta]$ representing the best options already secured for MAX and MIN respectively, we can identify subtrees that cannot possibly influence the final decision and **prune** them entirely.

- $\alpha$ is the best value MAX is guaranteed so far (updated at MAX nodes).
- $\beta$ is the best value MIN is guaranteed so far (updated at MIN nodes).
- We prune a branch when $\alpha \geq \beta$ — MIN would never allow MAX to reach this node.

Revisiting the example above, after evaluating X plays $(3,1)$ minimax sets $\alpha = 0$. When it then evaluates X plays $(3,3)$ and immediately finds value +1, it updates $\alpha = 1$. Any remaining unexplored siblings can be pruned since MAX already has a guaranteed +1. In the best case (well-ordered moves), alpha-beta pruning reduces the effective branching factor from $b$ to $\sqrt{b}$, roughly doubling the achievable search depth.

```pseudocode
function ALPHA_BETA(state, alpha, beta, is_max_turn):
    if state is terminal:
        return utility(state)

    if is_max_turn:
        value = -∞
        for each successor s of state:
            value = max(value, ALPHA_BETA(s, alpha, beta, false))
            alpha = max(alpha, value)
            if alpha >= beta:
                break   // β cutoff — MIN would avoid this branch
        return value
    else:
        value = +∞
        for each successor s of state:
            value = min(value, ALPHA_BETA(s, alpha, beta, true))
            beta = min(beta, value)
            if alpha >= beta:
                break   // α cutoff — MAX would avoid this branch
        return value
```

## Monte Carlo Tree Search

The final algorithm we will talk about is MCTS, a very powerful search algorithm that can be used in complex games and planning problems. The idea is simple: perform randomly simulated rollouts and use the results to determine the best move. For example, consider a chess position — this position is a single node in the tree. To find the best move, MCTS runs $N$ random simulations from that state as root node. By aggregating which moves tend to lead to wins versus losses, we identify the strongest candidate moves.

MCTS starts with a game tree that contains only the root node and builds the tree progressively. Every step of MCTS consists of four stages:

- **Selection:** We choose which child node to visit next. This choice is governed by the **UCB1 rule**, which balances exploiting the best-known move with exploring nodes that have been visited infrequently:

$$UCB1(v) = \frac{w_v}{n_v} + c \sqrt{\frac{\ln N}{n_v}}$$

where $w_v$ is the win count of node $v$, $n_v$ is its visit count, $N$ is the total simulations run so far, and $c$ is an exploration constant. We descend the tree by always selecting the child with the highest UCB1 score.

- **Expansion:** Once we reach a node that has unvisited children, we add one (or more) of those children as new nodes in the tree. This is how the tree grows over time.

- **Simulation:** Using the newly added node as the start state, we run a **random rollout** — both players take random moves until the game ends. This gives a fast, approximate signal of how promising this state is.

- **Backpropagation:** The result of the simulation (win = 1, loss = 0, draw = 0.5) is propagated back up the tree to the root. For each node on the path, we increment the visit count $n_v$ by 1 and the win count $w_v$ by the result. This updates UCB1 scores and steers future selection toward promising branches.

**Example 7: Tic-Tac-Toe with MCTS**

Suppose it is X's turn at the root and three moves are available: $(3,1)$, $(3,2)$, $(3,3)$.

After several iterations of MCTS the statistics might look like:

| Move    | Visits | Wins | Win Rate |
|---------|--------|------|----------|
| $(3,1)$ | 2      | 0.5  | 0.25     |
| $(3,2)$ | 2      | 0.5  | 0.25     |
| $(3,3)$ | 2      | 2.0  | 1.00     |

High UCB1 scores initially push the algorithm to explore each move at least once. As simulations accumulate, move $(3,3)$ — which leads to an immediate X win — consistently returns a reward of 1 and attracts more visits. After enough iterations, MCTS confidently selects $(3,3)$ as the best move.

The real power of MCTS emerges in games with enormous branching factors like Go, where the state space is too large for minimax with alpha-beta pruning to search deeply. MCTS does not need a hand-crafted evaluation function — the random rollout statistics serve as a built-in value estimate that improves with more simulations. This property, combined with the UCB1 exploration-exploitation balance, is what made MCTS the backbone of the AlphaGo system that first defeated world-champion Go players.
