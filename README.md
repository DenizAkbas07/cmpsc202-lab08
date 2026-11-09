# Lab 08: Navigating Weighted Graphs with Dijkstra's Algorithm

Name(s): Deniz Akbas 

## Introduction
You are already familiar with Breadth-First Search (BFS) and Depth-First Search (DFS) for exploring graphs. However, what happens when all edges are not created equal? In this lab, we will explore Dijkstra's Algorithm, which allows us to find the shortest path between two vertices in a *weighted* graph.

---

## Part 1: BFS with Weights

BFS is excellent for finding the shortest path in an unweighted graph. Let's see why it fails when weights are introduced. Consider the following weighted graph:

![Part 1 Graph](./part1.png)

**Questions:**
1. If you run a standard BFS starting at Node $A$, looking for Node $D$, which path will the BFS algorithm discover first? Why?
BFS finds the direct path A → D first. BFS explores level by level and ignores weights. D is one edge from A, so it’s reached before the three edge route.

2. What is the total weight of the path BFS found? What is the total weight of the alternative path? 
The BFS path A → D has weight 100. The alternative A → B → C → D has weight 2 + 3 + 4 = 9.

3. How does the definition of "shortest path" change when moving from unweighted graphs to weighted graphs?
In an unweighted graph, the shortest path is the one with the fewest edges. In a weighted graph, it’s the path with the smallest total edge weight, even if it uses more edges.

---

## Part 2: Dijkstra's Algorithm

Dijkstra's algorithm uses a greedy approach. It maintains a set of "visited" (finalized) nodes and an array of the shortest known distances to all other nodes. Consider the following weighted graph:

![Part 2 Graph](./part2.png)

**Instructions:**
Starting at Node $A$, trace Dijkstra's algorithm. Fill in the tracker table below. 
*   **Current Node:** The unvisited node with the smallest current distance.
*   **Neighbors to Check:** The neighbors of the current node whose distances might need updating.
*   **`dist` Array:** The current known shortest distance from $A$ to each node. Use $\infty$ for unreachable/unknown nodes.
*   **`visited` Set:** The set of nodes that have been finalized and whose shortest distances are known.

| Step | Current Node | Neighbors to Check | `dist` Map | `visited` Set | 
| :--- | :--- | :--- | :--- | :--- | 
| 0 | $A$ | $B$, $C$ | $A$:0, $B$:4, $C$:2 | { } | 
| 1 | C | B,D,E | A:0, B:3, C:2, D:10, E:12 | {A} | 
| 2 | B | D | A:0, B:3, C:2, D:10, E:12 | {A, C} | 
| 3 | D | E | A:0, B:3, C:2, D:8, E:10 | {A, C, B} | 
| 4 | E | none | A:0, B:3, C:2, D:8, E:10 | {A, C, B, D}  | 

**Conceptual Question:**
When we add a vertex to the "visited" set, why are we 100% certain we have found the absolute shortest path to that node? 
> Dijkstra always finalizes the unvisited node with the smallest tentative distance. Any other route to that node would have to pass through some other unvisited node, and that node’s distance is already at least as large. With non-negative weights, extending a route can only add cost, so no other route can be shorter.

---

## Part 3: Assumptions

Dijkstra's algorithm relies on an assumption about the weights. Let's see what happens when we break it. Consider the following weighted graph, which contains a negative edge:

![Part 3 Graph](./part3.png)

**Instructions:**
Trace Dijkstra's algorithm starting from Node $A$ to find the shortest path to Node $C$.

1. Which node does Dijkstra's algorithm visit first after $A$?
   > C. After relaxing A’s neighbors, dist is B:2 and C:1. C has the smaller distance, so it is visited first.

2. What will Dijkstra's algorithm declare as the shortest path to $C$, and what is its weight?
   > It declares the direct path A → C with weight 1. C is finalized right after A, so its distance is never updated again.

3. Look at the graph manually. What is the *actual* shortest path from $A$ to $C$, and what is its total weight? 
   > The actual shortest path is A → B → C with weight 2 + (-5) = -3, which is cheaper than the direct edge of 1.

4. **The Trap:** Explain exactly *why* Dijkstra's algorithm failed to find the correct shortest path in this scenario.
   > Dijkstra’s algorithm assumes all edge weights are non-negative. Under that assumption, once a node has the smallest distance among the unvisited nodes, no other route can beat it, because extending a route can only add cost. Here, C was finalized at distance 1 before B was explored. When B was visited later, the edge B → C (-5) offered a cheaper route of -3, but C was already in the visited set and is never updated. The negative edge made a longer-looking path cheaper, which broke the “visited means final” guarantee.

---

## Part 4: Implementing Dijkstra's Algorithm

In the real world, problems rarely look like neat mathematical graphs right away. You often have to translate real-world constraints into a standard graph format so that an algorithm like Dijkstra's can process it.

**The Scenario:**
You are building a navigation app for delivery robots on a college campus. 

* Moving between any two adjacent sidewalk intersections takes exactly $2$ minutes.

* Some paths have stairs. The robots *cannot* use stairs.

* Certain intersections are busy crosswalks. The robot must wait an additional $3$ minutes at these specific intersections before it can safely leave them and continue on its path.

**The Problem:**
Standard Dijkstra's algorithm only understands *edges* and *edge weights*. It does not natively understand "node weights" (like waiting at an intersection) or "impassable terrain".

**Questions:**

1. How should you represent the paths with stairs in your graph so the algorithm never tries to use them?

   > Leave stair paths out of the graph entirely. If two intersections are connected only by stairs, no edge is created between them. Dijkstra’s algorithm only moves along edges, so it can never route a robot over the stairs.

2. How can you account for the $3$-minute waiting time at busy crosswalk intersections using *only* edge weights? (Hint: Think about how you could modify the weights of edges connecting to that intersection).

   > Add the 3-minute wait to every edge that leaves a busy crosswalk, so those edges cost 5 minutes (2 + 3) and all other edges cost 2. Any path through a crosswalk then pays the wait exactly once, when the robot departs. Each sidewalk segment becomes two directed edges, since the penalty depends on which end the robot leaves from. All weights stay non-negative, so Dijkstra still works.