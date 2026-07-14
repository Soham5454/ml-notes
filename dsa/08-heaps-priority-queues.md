# 08 — Heaps / Priority Queues

Recap file 06's honest limitation note — hashing is bad at "efficiently find the min/max while data keeps changing." Heaps are built specifically for exactly that job.

## What a heap actually is

A heap is a specialized, COMPLETE binary tree (recap file 07) satisfying the **heap property**: in a MIN-heap, every parent is smaller than or equal to its children (so the smallest element is always at the root); in a MAX-heap, every parent is larger than or equal to its children.

Genuinely important distinction from a BST (file 07): a heap is only PARTIALLY ordered — it guarantees the root is the min/max, but says NOTHING about the relative order of siblings or across different subtrees. This weaker guarantee is precisely what makes heap operations faster/simpler than full BST balancing for the specific "give me the min/max" use case.

## Python's `heapq` — min-heap, built-in

```python
import heapq

heap = []
heapq.heappush(heap, 5)
heapq.heappush(heap, 1)
heapq.heappush(heap, 8)
heapq.heappush(heap, 3)

print(heap[0])            # 1 -- the minimum is ALWAYS at index 0, O(1) to peek
print(heapq.heappop(heap))  # 1, removes and returns the minimum, O(log n)
print(heap[0])              # 3 -- new minimum after the pop
```

`heapq` in Python only implements a MIN-heap directly — genuinely the most common "gotcha" for beginners is needing a MAX-heap, which Python doesn't provide natively.

## The max-heap trick — negate the values

```python
max_heap = []
for num in [5, 1, 8, 3]:
    heapq.heappush(max_heap, -num)      # push the NEGATIVE

largest = -heapq.heappop(max_heap)       # negate back when popping
print(largest)   # 8
```

Genuinely a real, commonly-used trick, not a hack to be embarrassed about — negating values to simulate a max-heap using a min-heap implementation is standard practice across competitive programming and real interview solutions.

## Why heap operations are O(log n) — the underlying array representation

```python
# A heap is conceptually a binary tree, but stored efficiently as a flat array
# For index i: left child = 2*i + 1, right child = 2*i + 2, parent = (i-1)//2

heap = [1, 3, 8, 5, 9]
# Tree shape:
#        1
#       / \
#      3   8
#     / \
#    5   9
```

Inserting or removing requires "bubbling" an element up or down the tree to restore the heap property — genuinely bounded by the tree's HEIGHT, which for a complete binary tree with `n` nodes is `log(n)` (recap file 07's height discussion) — precisely why heap push/pop are `O(log n)`, not `O(n)` or `O(1)`.

## Heapify — building a heap from an existing list, faster than repeated pushes

```python
arr = [5, 1, 8, 3, 9, 2]
heapq.heapify(arr)     # O(n), genuinely faster than n individual O(log n) pushes (which would be O(n log n))
print(arr[0])            # smallest element, now at index 0
```

Worth knowing this exists specifically because it's a genuinely common interview follow-up: "can you build the heap faster than pushing one at a time" — yes, `heapify` uses a smarter bottom-up approach that achieves O(n) instead of O(n log n).

## Kth largest/smallest element — the canonical heap use case

```python
def kth_largest(nums, k):
    heap = nums[:k]
    heapq.heapify(heap)          # min-heap of the first k elements

    for num in nums[k:]:
        if num > heap[0]:
            heapq.heapreplace(heap, num)   # pop smallest, push new -- O(log k), more efficient than separate pop+push

    return heap[0]

print(kth_largest([3, 2, 1, 5, 6, 4], 2))   # 5, the 2nd largest
```

Genuinely elegant approach: maintain a min-heap of SIZE k containing the k largest elements seen so far — the ROOT of this heap (the smallest of the k largest) is exactly the kth largest overall. `O(n log k)` time — notably better than sorting the whole array (`O(n log n)`) when `k` is small relative to `n`.

## Priority Queue — the more general concept a heap implements

A priority queue is an abstract data type where each element has a PRIORITY, and the element with the highest (or lowest) priority is always served first — regardless of insertion order, unlike file 05's plain FIFO queue.

```python
import heapq

# Storing (priority, task) tuples -- heapq compares tuples element by element,
# so it naturally sorts by priority first
tasks = []
heapq.heappush(tasks, (2, "write report"))
heapq.heappush(tasks, (1, "fix critical bug"))
heapq.heappush(tasks, (3, "reply to email"))

while tasks:
    priority, task = heapq.heappop(tasks)
    print(f"Priority {priority}: {task}")
# Priority 1: fix critical bug
# Priority 2: write report
# Priority 3: reply to email
```

Genuinely a real-world-relevant pattern — task schedulers, Dijkstra's algorithm (recap forward to file 16's graph algorithms), and A* pathfinding all lean on exactly this "always process the highest-priority item next" mechanism.

## Merging K sorted lists — a genuinely classic heap application

```python
def merge_k_sorted_lists(lists):
    heap = []
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst[0], i, 0))   # (value, list_index, element_index)

    result = []
    while heap:
        val, list_idx, elem_idx = heapq.heappop(heap)
        result.append(val)
        if elem_idx + 1 < len(lists[list_idx]):
            next_val = lists[list_idx][elem_idx + 1]
            heapq.heappush(heap, (next_val, list_idx, elem_idx + 1))

    return result

print(merge_k_sorted_lists([[1,4,5], [1,3,4], [2,6]]))
# [1, 1, 2, 3, 4, 4, 5, 6]
```

Genuinely worth noting the tuple trick here: including `list_idx` as a tiebreaker in the tuple avoids a crash if two elements have the EQUAL values (Python would otherwise try to compare `lst` objects directly if the first tuple elements tie, which can error). `O(n log k)` where `n` is total elements and `k` is number of lists — a real improvement over concatenating everything and sorting (`O(n log n)`) when `k` is small.

## Try this

```python
# Given a stream of numbers arriving one at a time, maintain the ability to return
# the MEDIAN of all numbers seen so far, after each new number arrives, in O(log n) per insert
# Hint: maintain TWO heaps -- a max-heap for the smaller half of numbers,
# a min-heap for the larger half -- keep them balanced in size, and the median is
# always derivable from their two roots
```

"Find median from a data stream" is genuinely a favorite hard-tier heap problem specifically because it requires combining TWO heaps cleverly — a strong test of whether the min-heap/max-heap mechanics from this file have properly clicked.

## What's next
File 09 — Graphs, generalizing trees (which are just a specific, constrained kind of graph — no cycles, one root) into the fully general structure behind networks, maps, dependency chains, and social connections.
