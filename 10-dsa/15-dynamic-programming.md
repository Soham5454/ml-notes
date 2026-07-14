# 15 — Dynamic Programming

Genuinely the topic most self-taught learners find hardest — and directly the fix for file 02's naive Fibonacci and file 14's coin-change counterexample, both flagged explicitly as needing this.

## The two properties DP requires — recap file 14's framework

1. **Overlapping subproblems** — the same smaller problem gets solved repeatedly (recap file 02's `fib_naive` recomputing `fib(3)` many times).
2. **Optimal substructure** — the optimal solution can be built from optimal solutions to subproblems (recap file 14's exact same term).

If BOTH hold, DP applies: solve each unique subproblem ONCE, store the result, reuse it instead of recomputing.

## Memoization (top-down) — recap file 02's fix, now fully explained

```python
def fib_memo(n, cache=None):
    if cache is None:
        cache = {}
    if n in cache:
        return cache[n]
    if n <= 1:
        return n
    cache[n] = fib_memo(n-1, cache) + fib_memo(n-2, cache)
    return cache[n]

print(fib_memo(50))   # instant -- fib_naive(50) would genuinely take an impractically long time
```

`O(n)` time now, down from `O(2^n)` (recap file 01/02) — genuinely just "recursion, but remember answers so you never solve the same subproblem twice."

## Tabulation (bottom-up) — the other genuinely standard DP approach

```python
def fib_tabulation(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

print(fib_tabulation(50))
```

Same `O(n)` time, but builds the answer ITERATIVELY from the base cases upward, rather than recursively from the top down. Genuinely a real, practical advantage: no recursion depth risk (recap file 02's call stack concern), and often easier to further optimize for space:

```python
def fib_optimized(n):          # O(1) SPACE -- only need the last 2 values, not the whole array
    if n <= 1:
        return n
    prev2, prev1 = 0, 1
    for _ in range(2, n + 1):
        prev2, prev1 = prev1, prev2 + prev1
    return prev1
```

## The DP problem-solving framework — the genuinely reusable process

1. Define what `dp[i]` (or `dp[i][j]`) MEANS in plain words.
2. Find the base case(s) — the smallest subproblems, solvable directly.
3. Find the recurrence — how does `dp[i]` relate to smaller/previous states?
4. Decide: memoization (top-down, natural if the recursive structure is clear) or tabulation (bottom-up, natural once the recurrence is clear)?
5. Optimize space if possible (recap `fib_optimized` above).

## Coin Change — properly solving file 14's greedy counterexample

```python
def coin_change_dp(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0                                  # base case: 0 coins needed for amount 0

    for a in range(1, amount + 1):
        for coin in coins:
            if coin <= a:
                dp[a] = min(dp[a], dp[a - coin] + 1)

    return dp[amount] if dp[amount] != float('inf') else -1

print(coin_change_dp([1, 3, 4], 6))   # 2 -- CORRECTLY finds 3+3, unlike file 14's greedy which gave 3
```

Genuinely satisfying to see this fix the exact case where greedy failed — `dp[a]` means "minimum coins needed to make amount `a`," and the recurrence tries EVERY coin as a possible "last coin used," taking the best result, rather than committing greedily to the biggest coin first.

## Longest Common Subsequence (LCS) — a genuinely classic 2D DP problem

```python
def lcs(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])

    return dp[m][n]

print(lcs("abcde", "ace"))   # 3 ("ace")
```

`dp[i][j]` means "length of the LCS between the first `i` characters of text1 and first `j` characters of text2" — genuinely the standard 2D DP template: build a grid, fill it based on a recurrence comparing characters, and the answer sits in the bottom-right corner. This exact pattern reappears (with small variations) in edit distance, longest common substring, and sequence alignment problems.

## 0/1 Knapsack — the other genuinely canonical DP template

```python
def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for w in range(capacity + 1):
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i-1][w], dp[i-1][w - weights[i-1]] + values[i-1])
            else:
                dp[i][w] = dp[i-1][w]

    return dp[n][capacity]

print(knapsack([1, 3, 4, 5], [1, 4, 5, 7], 7))   # 9
```

`dp[i][w]` means "maximum value achievable using the first `i` items, within weight capacity `w`" — at each item, the recurrence considers BOTH options (skip the item, or take it if it fits) and keeps the better one. Genuinely the template underlying an enormous family of "choose a subset under a constraint to maximize/minimize something" problems.

## Word Break — closing the loop from file 10's trie preview

```python
def word_break(s, word_dict):
    word_set = set(word_dict)          # recap file 06's O(1) hashing lookup
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True                          # base case: empty string is trivially "breakable"

    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break

    return dp[n]

print(word_break("catcard", ["cat", "card"]))   # True
```

Recap file 10's "try this" exercise, which solved this with plain recursion — that naive version genuinely re-explores overlapping subproblems (whether `s[j:]` is breakable gets recomputed repeatedly for the same `j`). This DP version fixes exactly that, `O(n²)` instead of exponential, using the same "define what dp[i] means" framework from this file.

## Try this

```python
# Solve "House Robber": given an array of home values, find the maximum sum obtainable
# by robbing houses, where no two ADJACENT houses can both be robbed
# Use the 5-step framework from this file explicitly:
# 1. Define dp[i] in words  2. Base case  3. Recurrence  4. Top-down or bottom-up  5. Optimize space
# Then attempt the naive RECURSIVE version first (no memoization) and observe how slow
# it gets for larger inputs, before adding memoization and comparing runtimes directly
```

Explicitly writing out all 5 framework steps (not just jumping to code) is genuinely the discipline that makes DP problems tractable under interview pressure — most people who struggle with DP are skipping straight to code without first nailing down exactly what `dp[i]` means.

## What's next
File 16 — Graph Algorithms (Dijkstra, Topological Sort, Union-Find, MST), extending file 09's BFS/DFS foundation into weighted shortest paths, dependency ordering, and connectivity problems.
