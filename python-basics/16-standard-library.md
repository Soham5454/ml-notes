# 16 — Useful Standard Library (LeetCode/ML relevant)

## collections module

### Counter — frequency counting
```python
from collections import Counter
c = Counter("mississippi")
c.most_common(2)     # [('i', 4), ('s', 4)]
```

### defaultdict — dict with automatic default values
```python
from collections import defaultdict
d = defaultdict(int)          # default value 0 for missing keys
d["a"] += 1                     # no KeyError even though "a" wasn't set first

graph = defaultdict(list)         # default value [] for missing keys
graph["A"].append("B")
```

### deque — double-ended queue, O(1) append/pop from both ends
```python
from collections import deque
dq = deque([1, 2, 3])
dq.append(4)          # add to right -> [1, 2, 3, 4]
dq.appendleft(0)         # add to left  -> [0, 1, 2, 3, 4]
dq.pop()                    # remove from right
dq.popleft()                  # remove from left
```
`deque` is the standard choice for BFS (queue) problems — regular lists have O(n) `pop(0)`, deque has O(1) `popleft()`.

### OrderedDict — dict that remembers insertion order (less needed since 3.7, but still used)
```python
from collections import OrderedDict
od = OrderedDict()
od["a"] = 1
od["b"] = 2
```

## heapq — priority queues / heaps
```python
import heapq
heap = [5, 1, 3]
heapq.heapify(heap)         # converts list into a valid min-heap in place
heapq.heappush(heap, 2)        # add element
heapq.heappop(heap)               # remove and return SMALLEST element

# For a max-heap, negate values on the way in/out:
heapq.heappush(heap, -5)
-heapq.heappop(heap)
```
Common in "top-K", Dijkstra's algorithm, and merge-K-lists problems.

## itertools — combinatorics and iteration tools
```python
import itertools

list(itertools.permutations([1,2,3]))          # all orderings
list(itertools.combinations([1,2,3], 2))          # all unique pairs
list(itertools.product([1,2], [3,4]))                # cartesian product
list(itertools.accumulate([1,2,3,4]))                   # running totals: [1,3,6,10]
```

## math module
```python
import math
math.sqrt(16)          # 4.0
math.floor(3.7)          # 3
math.ceil(3.2)              # 4
math.gcd(12, 18)               # 6
math.factorial(5)                # 120
math.log(100, 10)                  # 2.0  (log base 10 of 100)
math.inf                              # infinity -> useful for initializing min-comparisons
```

## bisect — binary search on sorted lists
```python
import bisect
lst = [1, 3, 4, 7]
bisect.bisect_left(lst, 4)      # 2  -> leftmost insertion point to keep list sorted
bisect.insort(lst, 5)              # inserts 5 in the correct sorted position
```

## string module (constants, not to be confused with str type)
```python
import string
string.ascii_lowercase     # 'abcdefghijklmnopqrstuvwxyz'
string.ascii_uppercase       # 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
string.digits                  # '0123456789'
```

## sys module — for LeetCode-style fast I/O and recursion limits
```python
import sys
sys.setrecursionlimit(10000)     # increase default recursion depth (default ~1000)
input = sys.stdin.readline          # faster input reading for competitive programming
```

## functools
```python
from functools import lru_cache

@lru_cache(maxsize=None)     # memoization decorator — caches function results
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)
```
`lru_cache` turns naive exponential recursion (like plain Fibonacci) into efficient memoized recursion with one line — extremely useful for DP problems.
