# 09 — Graphs: Representation, BFS & DFS

Recap file 07's tree definition — a tree is genuinely just a constrained graph (no cycles, exactly one root, every node has exactly one parent). Graphs drop all those constraints: any node can connect to any other, cycles are allowed, and there's no single "root."

## Representing a graph — two genuinely standard approaches

```python
# Adjacency List -- the most common representation, especially for sparse graphs
graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D'],
    'C': ['A', 'D'],
    'D': ['B', 'C', 'E'],
    'E': ['D']
}

# Adjacency Matrix -- better for dense graphs, or when O(1) edge-existence checks matter
#      A  B  C  D  E
# A  [ 0, 1, 1, 0, 0 ]
# B  [ 1, 0, 0, 1, 0 ]
# ...
```

Genuinely important tradeoff to know: adjacency list uses `O(V + E)` space (V=vertices, E=edges) and is efficient for sparse graphs (few connections relative to possible connections); adjacency matrix uses `O(V^2)` space regardless of actual edge count, but gives O(1) "is there an edge between X and Y" checks. Most interview problems use adjacency lists.

## Directed vs undirected, weighted vs unweighted

```python
# Undirected: edge A-B means both A->B and B->A exist (the dict example above)

# Directed: edges have a specific direction
directed_graph = {
    'A': ['B'],       # A -> B, but NOT necessarily B -> A
    'B': ['C'],
    'C': []
}

# Weighted: edges have an associated cost/weight (used heavily in file 16's Dijkstra)
weighted_graph = {
    'A': [('B', 4), ('C', 1)],    # (neighbor, weight)
    'B': [('D', 2)],
    'C': [('B', 1), ('D', 5)],
    'D': []
}
```

## Depth-First Search (DFS) — recursion or an explicit stack

```python
def dfs_recursive(graph, node, visited=None):
    if visited is None:
        visited = set()
    visited.add(node)
    print(node, end=' ')
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited)
    return visited

dfs_recursive(graph, 'A')   # A B D C E (order depends on adjacency list ordering)
```

```python
def dfs_iterative(graph, start):
    visited = set()
    stack = [start]           # recap file 05's stack -- DFS is genuinely just this

    while stack:
        node = stack.pop()
        if node not in visited:
            visited.add(node)
            print(node, end=' ')
            for neighbor in graph[node]:
                if neighbor not in visited:
                    stack.append(neighbor)

dfs_iterative(graph, 'A')
```

Genuinely the key insight worth internalizing: DFS explores as DEEP as possible down one path before backtracking — recursion handles this naturally because the call stack itself IS the stack being used (recap file 02's call stack visualization). The iterative version makes that stack explicit instead of relying on Python's own recursion.

## Breadth-First Search (BFS) — level by level, using a queue

```python
from collections import deque

def bfs(graph, start):
    visited = set([start])
    queue = deque([start])      # recap file 05's queue -- BFS is genuinely just this
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    return order

print(bfs(graph, 'A'))   # ['A', 'B', 'C', 'D', 'E']
```

Recap file 07's level-order traversal — that WAS BFS, applied to a tree. This is the exact same mechanism generalized to any graph. The critical property genuinely worth remembering: **BFS visits nodes in order of increasing distance from the start** — this is precisely why BFS is the go-to algorithm for "shortest path in an UNWEIGHTED graph" problems.

## DFS vs BFS — when to reach for which, honestly

| | DFS | BFS |
|---|---|---|
| Underlying structure | Stack (or recursion) | Queue |
| Explores | Deep, then backtracks | Level by level, wide first |
| Shortest path (unweighted) | Does NOT guarantee shortest path | GUARANTEES shortest path |
| Memory | Can be more memory-efficient (just the current path) | Can use more memory (entire frontier level) |
| Good for | Detecting cycles, topological sort (file 16), exploring all paths, backtracking-style problems | Shortest path, "minimum steps" problems, level-by-level processing |

## Connected components — a genuinely common graph problem

```python
def count_connected_components(graph, all_nodes):
    visited = set()
    components = 0

    for node in all_nodes:
        if node not in visited:
            components += 1
            dfs_recursive(graph, node, visited)    # mark this whole component visited

    return components
```

Genuinely a common real-world framing too — social network "friend groups," clusters of related web pages, or network reachability problems all reduce to counting connected components.

## Cycle detection — different approaches for directed vs undirected

```python
def has_cycle_undirected(graph, node, visited, parent):
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            if has_cycle_undirected(graph, neighbor, visited, node):
                return True
        elif neighbor != parent:      # visited AND not where we just came from -- genuine cycle
            return True
    return False
```

The `parent` check matters genuinely for undirected graphs — without it, every edge would be falsely flagged as a cycle, since going A→B then checking B's neighbors would immediately "see" A again (the edge back the way it came, not an actual cycle). Directed graph cycle detection needs a different approach (tracking nodes currently "in progress" on the recursion stack) — covered properly alongside topological sort in file 16.

## Grid problems as graphs — a genuinely common disguised-graph pattern

```python
def num_islands(grid):
    if not grid:
        return 0
    rows, cols = len(grid), len(grid[0])
    visited = set()

    def dfs(r, c):
        if (r < 0 or r >= rows or c < 0 or c >= cols or
            (r, c) in visited or grid[r][c] == '0'):
            return
        visited.add((r, c))
        for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:    # up, down, left, right
            dfs(r + dr, c + dc)

    islands = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1' and (r, c) not in visited:
                islands += 1
                dfs(r, c)

    return islands
```

Genuinely important pattern-recognition skill: a 2D grid where cells connect to their neighbors IS a graph — each cell is a node, adjacency is implicit (up/down/left/right), and DFS/BFS apply directly, just using `(row, col)` tuples as node identifiers (recap file 06's tuple-as-dict-key note) instead of named nodes like 'A'/'B'.

## Try this

```python
# Given a graph representing course prerequisites (course A must be taken before course B),
# determine if it's possible to finish all courses (i.e. the graph has NO cycle)
# This is genuinely the exact real-world motivation for topological sort, covered fully in file 16
# For now, solve it using ONLY cycle detection on a directed graph (a genuine preview exercise)
```

Course scheduling is genuinely one of the most commonly asked graph problems precisely because it maps a real, intuitive scenario (can't take a course before its prerequisite) onto pure cycle-detection logic — good motivation heading into file 16's full topological sort treatment.

## What's next
File 10 — Tries, a specialized tree structure built for string/prefix problems — genuinely useful for autocomplete, spell-checkers, and word-search problems, and a nice callback to the NLP tokenization concepts from the PyTorch/Hugging Face notes.
