# 02 — Recursion & Backtracking

Recap file 01's `mystery3` example — that was recursion, used without naming it yet. This file makes it explicit, since it's genuinely the mental model behind trees (07), graphs (09), and dynamic programming (15).

## What recursion actually is

A function that calls itself, working on a SMALLER version of the same problem each time, until it hits a **base case** simple enough to answer directly.

```python
def factorial(n):
    if n <= 1:               # base case -- stops the recursion
        return 1
    return n * factorial(n - 1)   # recursive case -- smaller version of the same problem

print(factorial(5))   # 120
```

Every recursive function needs genuinely both parts: a base case (or it never stops — infinite recursion, eventually a `RecursionError` in Python) and a recursive case that makes REAL progress toward the base case.

## Visualizing the call stack — the piece that makes recursion click

```python
def factorial_traced(n, depth=0):
    print("  " * depth + f"factorial({n}) called")
    if n <= 1:
        print("  " * depth + f"factorial({n}) returns 1")
        return 1
    result = n * factorial_traced(n - 1, depth + 1)
    print("  " * depth + f"factorial({n}) returns {result}")
    return result

factorial_traced(4)
```

Recap file 01's space complexity note — each recursive call adds a new "frame" to the call stack, genuinely consuming real memory (`O(n)` space for `n` deep recursion) until base cases are hit and the stack starts "unwinding," returning values back up through each waiting call. Python's default recursion limit (~1000 by default) exists precisely because unbounded recursion depth is a real risk.

## Fibonacci — the classic example that ALSO reveals recursion's biggest weakness

```python
def fib_naive(n):
    if n <= 1:
        return n
    return fib_naive(n-1) + fib_naive(n-2)

print(fib_naive(10))   # 55, correct
```

Recap file 01's `O(2^n)` complexity class — this naive version is genuinely exponential, because it RECOMPUTES the same subproblems repeatedly (e.g. `fib(5)` and `fib(4)` both independently recompute `fib(3)`). This exact inefficiency is what file 15 (Dynamic Programming) fixes via memoization — flagging this now so DP doesn't feel like an unrelated new topic later, it's directly solving THIS problem.

```python
# Quick preview of the fix (full treatment in file 15)
def fib_memo(n, cache={}):
    if n in cache:
        return cache[n]
    if n <= 1:
        return n
    cache[n] = fib_memo(n-1, cache) + fib_memo(n-2, cache)
    return cache[n]

print(fib_memo(50))   # instant, vs fib_naive(50) which would take genuinely forever
```

## Recursion vs iteration — when to actually reach for recursion

```python
# Both compute the same thing -- sum of 1 to n
def sum_iterative(n):
    total = 0
    for i in range(1, n+1):
        total += i
    return total

def sum_recursive(n):
    if n == 0:
        return 0
    return n + sum_recursive(n - 1)
```

Genuinely honest take: for simple linear accumulation like this, iteration is more memory-efficient (O(1) space vs recursion's O(n) call stack) and just as readable — recursion isn't automatically "better." Recursion earns its keep specifically when the PROBLEM STRUCTURE is naturally recursive/tree-shaped: traversing nested structures (trees, file directories, nested lists), or when a problem naturally breaks into smaller identical subproblems (divide and conquer, backtracking, many DP problems).

## Backtracking — recursion with "try, then undo if it doesn't work"

Backtracking builds a solution incrementally, and when a choice leads to a dead end, it "backtracks" — undoes that choice and tries a different one. Genuinely the pattern behind subset/permutation generation, N-Queens, Sudoku solvers, and maze pathfinding.

```python
def generate_subsets(nums):
    result = []

    def backtrack(start, current_subset):
        result.append(current_subset[:])    # record a COPY of the current state as a valid subset

        for i in range(start, len(nums)):
            current_subset.append(nums[i])    # CHOOSE
            backtrack(i + 1, current_subset)   # EXPLORE
            current_subset.pop()                # UN-CHOOSE (the actual "backtrack" step)

    backtrack(0, [])
    return result

print(generate_subsets([1, 2, 3]))
# [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

The **choose → explore → un-choose** pattern above is genuinely THE template for almost every backtracking problem — worth memorizing as a rhythm, the same way file 07 of the PyTorch folder flagged the five-step training loop rhythm.

## Permutations — the other classic backtracking template

```python
def generate_permutations(nums):
    result = []

    def backtrack(current, remaining):
        if not remaining:
            result.append(current[:])
            return
        for i in range(len(remaining)):
            current.append(remaining[i])
            backtrack(current, remaining[:i] + remaining[i+1:])
            current.pop()

    backtrack([], nums)
    return result

print(generate_permutations([1, 2, 3]))
# [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]
```

## A genuinely classic backtracking problem — N-Queens (simplified check)

```python
def solve_n_queens_count(n):
    def is_safe(row, col, placed_cols):
        for prev_row, prev_col in enumerate(placed_cols):
            if prev_col == col or abs(prev_col - col) == abs(prev_row - row):
                return False
        return True

    def backtrack(row, placed_cols):
        if row == n:
            return 1
        count = 0
        for col in range(n):
            if is_safe(row, col, placed_cols):
                placed_cols.append(col)      # CHOOSE
                count += backtrack(row + 1, placed_cols)   # EXPLORE
                placed_cols.pop()              # UN-CHOOSE
        return count

    return backtrack(0, [])

print(solve_n_queens_count(4))   # 2 -- the classic 4-Queens problem has exactly 2 solutions
```

Same choose-explore-un-choose template as the subsets example, just with an `is_safe` check gating which choices are even allowed — genuinely this template, once internalized, makes a huge fraction of "hard" backtracking interview problems feel like small variations on the same idea rather than each needing a fresh approach.

## Try this

```python
# Write a recursive function to reverse a string WITHOUT using slicing ([::-1]) or a loop
# Then write a backtracking function to generate all valid combinations of balanced
# parentheses for n pairs (e.g. n=2 -> ["(())", "()()"])
# Use the choose-explore-un-choose template from the subsets example as a starting structure
```

The balanced-parentheses problem is genuinely one of the most commonly asked backtracking interview questions — working through it using the exact template just learned is the clearest proof the pattern has actually been internalized, not just read.

## What's next
File 03 — Arrays & Strings, the most fundamental data structures, where two pointers and sliding window techniques (fully covered in file 13) start showing up constantly.
