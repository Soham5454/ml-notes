# 16 — Graph Algorithms (Dijkstra, Topological Sort, Union-Find, MST)

Recap file 09's BFS/DFS foundation and its "shortest path in unweighted graphs" note — this file extends that into WEIGHTED shortest paths, dependency ordering, and connectivity problems.

## Dijkstra's Algorithm — shortest path in a weighted graph

Recap file 09's honest limitation: BFS guarantees shortest path only for UNWEIGHTED graphs. Dijkstra fixes this for weighted graphs (with non-negative weights), using file 08's heap directly.

```python
import heapq

def dijkstra(graph, start):          # graph: {node: [(neighbor, weight), ...]}
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    pq = [(0, start)]                   # (distance, node) -- recap file 08's priority queue

    while pq:
        current_dist, node = heapq.heappop(pq)

        if current_dist > distances[node]:      # stale entry, already found a better path
            continue

        for neighbor, weight in graph[node]:
            distance = current_dist + weight
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                heapq.heappush(pq, (distance, neighbor))

    return distances

graph = {
    'A': [('B', 4), ('C', 1)],
    'B': [('D', 1)],
    'C': [('B', 2), ('D', 5)],
    'D': []
}
print(dijkstra(graph, 'A'))   # {'A': 0, 'B': 3, 'C': 1, 'D': 4}
```

Genuinely the core insight: always process the CLOSEST unvisited node next (recap file 08's priority queue task-scheduling example — same "always handle highest priority" mechanism), and whenever a shorter path to a neighbor is found through the current node, update it. `O((V+E) log V)` with a heap — the `log V` factor comes directly from file 08's heap push/pop cost.

Important honest limitation worth flagging clearly: Dijkstra does NOT work correctly with NEGATIVE edge weights — a genuinely common interview follow-up question, and the reason algorithms like Bellman-Ford exist for graphs where negative weights are possible (filing that away as a "know it exists" topic, not deep-diving here).

## Topological Sort — ordering with dependencies

Recap file 09's course-scheduling "try this" exercise — topological sort is precisely the tool that both detects cycles AND produces a valid ordering when none exist.

```python
from collections import deque

def topological_sort(graph, num_nodes):          # Kahn's algorithm, BFS-based
    in_degree = [0] * num_nodes
    for node in graph:
        for neighbor in graph[node]:
            in_degree[neighbor] += 1

    queue = deque([n for n in range(num_nodes) if in_degree[n] == 0])
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    if len(order) != num_nodes:
        return []                     # a cycle exists -- not all nodes could be ordered
    return order

course_graph = {0: [1], 1: [2], 2: [3], 3: []}
print(topological_sort(course_graph, 4))   # [0, 1, 2, 3]
```

Genuinely elegant mechanism: `in_degree` tracks how many prerequisites each node still has unmet. Nodes with ZERO remaining prerequisites go into the queue — process them, then decrement their neighbors' in-degrees, and any neighbor that reaches zero becomes ready too. If the final `order` doesn't include every node, some nodes never reached in-degree zero — meaning a CYCLE exists (a dependency loop that can never be satisfied), directly answering file 09's course-scheduling question with a genuinely complete algorithm rather than just a yes/no cycle check.

## Union-Find (Disjoint Set Union) — genuinely efficient connectivity tracking

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])    # path compression -- genuinely key optimization
        return self.parent[x]

    def union(self, x, y):
        root_x, root_y = self.find(x), self.find(y)
        if root_x == root_y:
            return False                # already connected -- this edge would form a cycle

        if self.rank[root_x] < self.rank[root_y]:           # union by rank -- keeps trees shallow
            root_x, root_y = root_y, root_x
        self.parent[root_y] = root_x
        if self.rank[root_x] == self.rank[root_y]:
            self.rank[root_x] += 1
        return True

uf = UnionFind(6)
uf.union(0, 1)
uf.union(1, 2)
print(uf.find(0) == uf.find(2))   # True -- 0 and 2 are connected through 1
print(uf.find(0) == uf.find(3))   # False -- 3 is in a different component
```

**Path compression** (in `find`) and **union by rank** (in `union`) are genuinely the two optimizations that make Union-Find nearly O(1) per operation in practice (technically O(α(n)), where α is the inverse Ackermann function — grows SO slowly it's effectively constant for any realistic input size). Genuinely the standard tool for "are these two things connected" and "detect a cycle while building a graph incrementally" problems — an alternative to file 09's DFS-based cycle detection, often cleaner for problems framed around incrementally adding edges.

## Minimum Spanning Tree (MST) — Kruskal's Algorithm, using Union-Find directly

An MST connects ALL nodes in a weighted graph using the MINIMUM total edge weight, with no cycles.

```python
def kruskal_mst(n, edges):          # edges: [(weight, u, v), ...]
    edges = sorted(edges)             # greedy: consider cheapest edges first (recap file 14's greedy logic)
    uf = UnionFind(n)
    mst_weight = 0
    mst_edges = []

    for weight, u, v in edges:
        if uf.union(u, v):              # only add the edge if it DOESN'T form a cycle
            mst_weight += weight
            mst_edges.append((u, v, weight))

    return mst_weight, mst_edges

edges = [(1, 0, 1), (4, 0, 2), (3, 1, 2), (2, 1, 3), (5, 2, 3)]
weight, tree_edges = kruskal_mst(4, edges)
print(weight, tree_edges)   # 6, using the 3 cheapest non-cycle-forming edges
```

Genuinely a beautiful combination: file 14's greedy philosophy (always take the cheapest available edge) proven CORRECT here (unlike the coin-change counterexample) specifically because MST construction satisfies both the greedy-choice and optimal-substructure properties — and Union-Find is precisely the efficient tool for the "does adding this edge create a cycle" check that greedy edge selection needs at every step.

## The honest map of which structure/algorithm solves which graph question

| Question | Tool |
|---|---|
| Shortest path, unweighted | BFS (file 09) |
| Shortest path, weighted, non-negative | Dijkstra (this file) |
| Shortest path, weighted, negative edges allowed | Bellman-Ford (know it exists, not covered in depth) |
| Valid ordering respecting dependencies | Topological Sort (this file) |
| Are these two nodes connected / will this edge create a cycle | Union-Find (this file) |
| Cheapest way to connect ALL nodes | MST — Kruskal's (this file) or Prim's (know it exists) |
| Explore/visit everything reachable | BFS or DFS (file 09) |

## Try this

```python
# Given a list of (city1, city2, cost) flight routes, and a starting city, find the
# cheapest cost to reach EVERY other reachable city using Dijkstra
# Then: given the same flight network, determine the minimum total cost to connect
# ALL cities (ignoring direction, treating cost as an undirected edge weight) using Kruskal's MST
# Compare the two answers and reason about WHY they're solving genuinely different questions
# despite using similar-looking weighted graph inputs
```

Explicitly contrasting Dijkstra's output (cheapest path FROM one specific start to every node) against Kruskal's output (cheapest way to connect ALL nodes to each other, not from any particular start) is genuinely the clearest way to avoid confusing these two algorithms, which is a common mix-up when first learning both.

## What's next
File 17 — Bit Manipulation, a genuinely different toolkit operating directly on binary representations — used for extreme space/speed optimizations and a distinct, recognizable category of interview questions.
