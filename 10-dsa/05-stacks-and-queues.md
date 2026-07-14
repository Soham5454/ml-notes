# 05 — Stacks & Queues

Two genuinely simple structures with an outsized number of real applications — DFS (file 09) uses a stack, BFS (file 09) uses a queue, and both reappear across a huge fraction of the rest of this roadmap.

## Stack — Last In, First Out (LIFO)

```python
stack = []

stack.append(1)     # push
stack.append(2)
stack.append(3)
print(stack.pop())    # 3 -- the LAST thing pushed is the FIRST thing popped
print(stack[-1])       # peek at the top without removing it -- 2
```

Python lists work perfectly as stacks — `.append()` and `.pop()` from the END are both genuinely O(1), recap file 03's array cost notes.

## The classic mental model — a stack of plates

Genuinely the easiest way to remember LIFO: you can only add or remove the TOP plate from a stack of plates. Function call stacks (recap file 02's recursion notes) work exactly this way — each function call gets "pushed," and when it returns, it gets "popped," always from the top.

## Valid parentheses — the single most common stack interview question

```python
def is_valid_parentheses(s):
    stack = []
    pairs = {')': '(', ']': '[', '}': '{'}

    for char in s:
        if char in '([{':
            stack.append(char)
        elif char in ')]}':
            if not stack or stack.pop() != pairs[char]:
                return False

    return len(stack) == 0   # everything must have been matched and popped

print(is_valid_parentheses("({[]})"))   # True
print(is_valid_parentheses("({[})"))     # False
```

Genuinely the canonical use case for a stack — matching/nesting problems where the MOST RECENT open item must be the FIRST one closed, exactly LIFO behavior.

## Monotonic stack — a genuinely high-value, less obvious pattern

A monotonic stack maintains elements in increasing (or decreasing) order, popping elements that violate that order as new ones come in. Used to solve "next greater element" style problems efficiently.

```python
def next_greater_element(arr):
    result = [-1] * len(arr)
    stack = []   # stores INDICES, maintains decreasing values from bottom to top

    for i in range(len(arr)):
        while stack and arr[stack[-1]] < arr[i]:
            prev_index = stack.pop()
            result[prev_index] = arr[i]     # arr[i] is the "next greater" for prev_index
        stack.append(i)

    return result

print(next_greater_element([2, 1, 2, 4, 3]))   # [4, 2, 4, -1, -1]
```

`O(n)` time — genuinely surprising the first time it's seen, since it LOOKS like it should be `O(n^2)` (checking every element against every future element). The trick: each element gets pushed and popped from the stack AT MOST once across the whole run, which is precisely what keeps total work linear.

## Queue — First In, First Out (FIFO)

```python
from collections import deque

queue = deque()
queue.append(1)      # enqueue
queue.append(2)
queue.append(3)
print(queue.popleft())   # 1 -- the FIRST thing added is the FIRST thing removed
```

Genuinely important practical note: use `collections.deque`, NOT a plain Python list, for queue behavior. `list.pop(0)` is `O(n)` (recap file 03's shifting cost), while `deque.popleft()` is `O(1)` — a deque is implemented as a doubly-linked structure (recap file 04) internally, specifically to make BOTH ends fast.

## The mental model — a line at a ticket counter

FIFO, genuinely the opposite ordering behavior from a stack — first person in line is the first one served, no matter how many people join after them.

## Queue in BFS — a direct preview of file 09

```python
from collections import deque

def bfs_preview(graph, start):
    visited = set([start])
    queue = deque([start])
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    return order
```

Genuinely the exact mechanism file 09 builds on fully — BFS explores level by level specifically BECAUSE a queue processes things in the order they were discovered, guaranteeing all nodes at distance 1 get processed before any node at distance 2.

## Deque as a double-ended structure — beyond just queue use

```python
dq = deque([1, 2, 3])
dq.appendleft(0)     # O(1) -- add to front
dq.append(4)           # O(1) -- add to back
dq.popleft()             # O(1) -- remove from front
dq.pop()                  # O(1) -- remove from back
```

A `deque` genuinely supports O(1) operations at BOTH ends — useful beyond plain queues, notably for the **monotonic deque** technique (used in sliding window maximum problems, an extension of file 03's sliding window pattern) and as an efficient stack alternative too.

## Implementing a queue using two stacks — a genuinely classic exercise

```python
class QueueUsingStacks:
    def __init__(self):
        self.stack_in = []
        self.stack_out = []

    def enqueue(self, value):
        self.stack_in.append(value)

    def dequeue(self):
        if not self.stack_out:
            while self.stack_in:                       # move everything, reversing order
                self.stack_out.append(self.stack_in.pop())
        return self.stack_out.pop()
```

Genuinely a good exercise for understanding both structures deeply — moving elements from `stack_in` to `stack_out` REVERSES their order once, which converts LIFO behavior into FIFO behavior. Amortized O(1) per operation, even though individual `dequeue` calls occasionally do O(n) work during the transfer.

## Try this

```python
# Implement a function that evaluates a simple postfix expression (Reverse Polish Notation)
# using a stack, e.g. "3 4 +" -> 7, "5 1 2 + 4 * + 3 -" -> 14
# Rule: read tokens left to right; numbers get pushed; operators pop the top two numbers,
# apply the operator, and push the result back
```

Postfix expression evaluation is genuinely a classic, satisfying stack exercise — and it's directly related to how calculators and some compilers actually evaluate expressions internally, not just an arbitrary interview puzzle.

## What's next
File 06 — Hashing, the technique behind Python's `dict`/`set`, and genuinely the single highest-leverage tool for converting O(n) or O(n²) brute-force solutions into O(n) ones.
