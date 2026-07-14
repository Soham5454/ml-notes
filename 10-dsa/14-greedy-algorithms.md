# 14 — Greedy Algorithms

A genuinely different problem-solving philosophy from everything so far: at every step, make the choice that looks BEST RIGHT NOW, without reconsidering it later — no backtracking (file 02), no exploring alternatives. The hard part isn't implementing greedy algorithms, it's knowing WHEN greedy actually gives the correct answer.

## The core idea, with a simple example

```python
def coin_change_greedy(coins, amount):
    coins = sorted(coins, reverse=True)
    count = 0
    for coin in coins:
        while amount >= coin:
            amount -= coin
            count += 1
    return count if amount == 0 else -1

print(coin_change_greedy([25, 10, 5, 1], 63))   # 6 (2 quarters, 1 dime, 3 pennies) -- correct for THIS coin system
```

Genuinely important, commonly-missed caveat: this greedy approach works for US coin denominations, but is NOT guaranteed to work for arbitrary coin systems.

```python
print(coin_change_greedy([1, 3, 4], 6))   # greedy gives 4+1+1 = 3 coins
# but the OPTIMAL answer is 3+3 = 2 coins -- greedy gets this WRONG
```

This is genuinely the single most important lesson of this whole file: **greedy is fast when it works, but it doesn't always give the optimal answer** — proving WHEN it's safe to use greedy (versus needing file 15's dynamic programming instead) is a real skill, not just a formality.

## When greedy is provably correct — the two properties to check

1. **Greedy choice property** — a globally optimal solution can be reached by making a locally optimal choice at each step, without needing to reconsider past choices.
2. **Optimal substructure** — an optimal solution to the problem contains optimal solutions to its subproblems (recap this term, it reappears identically in file 15's DP discussion — genuinely the same structural property, but DP exploits it by trying multiple options and remembering results, while greedy exploits it by committing to one choice immediately).

Genuinely, in an interview setting, formally proving these properties is rare — but being able to construct a COUNTEREXAMPLE (like the `[1,3,4]` coins case above) is a realistic and valuable skill for justifying "greedy won't work here, we need DP instead."

## Activity Selection — a genuinely classic, PROVABLY correct greedy problem

```python
def max_activities(activities):          # list of (start, end) tuples
    activities = sorted(activities, key=lambda a: a[1])   # sort by END time -- the key greedy insight
    count = 1
    last_end = activities[0][1]

    for start, end in activities[1:]:
        if start >= last_end:
            count += 1
            last_end = end

    return count

print(max_activities([(1,3), (2,4), (3,5), (0,6), (5,7), (8,9), (5,9)]))   # 4
```

Genuinely worth understanding WHY sorting by END time (not start time) is the correct greedy choice: picking the activity that finishes EARLIEST always leaves the MAXIMUM possible remaining time for future activities — this can actually be proven correct via an "exchange argument" (any other valid solution can be transformed into the greedy solution without making it worse), which is the standard way greedy algorithms get formally justified.

## Jump Game — a genuinely common greedy interview question

```python
def can_reach_end(nums):
    max_reach = 0
    for i, num in enumerate(nums):
        if i > max_reach:            # this position is unreachable
            return False
        max_reach = max(max_reach, i + num)
    return True

print(can_reach_end([2, 3, 1, 1, 4]))   # True
print(can_reach_end([3, 2, 1, 0, 4]))    # False -- get stuck at index 3
```

Greedy insight here: track the FARTHEST reachable index seen so far, updating it as we go — genuinely no need to try every possible jump sequence (which would be exponential, recap file 01), because the farthest-reach value alone captures everything needed to know whether the end is reachable.

## Gas Station — a genuinely elegant greedy insight

```python
def can_complete_circuit(gas, cost):
    if sum(gas) < sum(cost):
        return -1                     # total gas must be enough overall

    total = 0
    start = 0
    for i in range(len(gas)):
        total += gas[i] - cost[i]
        if total < 0:                   # can't reach the NEXT station from current start
            start = i + 1                  # so the start CAN'T be any station up to and including i either
            total = 0

    return start
```

Genuinely elegant, non-obvious insight worth appreciating: if the journey fails partway from some starting point, NONE of the stations between the original start and the failure point could have worked as a starting point either (they'd all fail at the same spot or earlier) — this lets the algorithm skip checking them individually, achieving `O(n)` instead of the naive `O(n²)` (trying every possible starting station from scratch).

## Huffman Coding — a genuinely real-world greedy application (compression)

```python
import heapq          # recap file 08's heap

def huffman_frequencies(text):
    from collections import Counter
    freq = Counter(text)
    heap = [[weight, [char, ""]] for char, weight in freq.items()]
    heapq.heapify(heap)

    while len(heap) > 1:
        lo = heapq.heappop(heap)
        hi = heapq.heappop(heap)
        for pair in lo[1:]:
            pair[1] = '0' + pair[1]
        for pair in hi[1:]:
            pair[1] = '1' + pair[1]
        heapq.heappush(heap, [lo[0] + hi[0]] + lo[1:] + hi[1:])

    return sorted(heapq.heappop(heap)[1:], key=lambda p: (len(p[-1]), p))

codes = huffman_frequencies("abracadabra")
for char, code in codes:
    print(char, code)
```

Genuinely a real compression algorithm used in actual file formats (ZIP, JPEG) — greedily combines the TWO least-frequent items repeatedly (using file 08's min-heap for efficient "find the two smallest" access), which provably produces an optimal prefix-free encoding, giving frequent characters shorter codes and rare characters longer ones.

## Greedy vs DP — the honest decision framework to carry forward

```
Ask: "If I make the locally-best choice now, could a DIFFERENT choice now ever lead
to a BETTER overall outcome later?"

If NO (the greedy choice never needs reconsidering) -> Greedy works, use it, it's faster.
If YES (an earlier suboptimal-looking choice could pay off later) -> Need DP (file 15)
to properly explore/remember multiple possibilities instead of committing early.
```

Genuinely the coin change example (`[1,3,4]`, target 6) from earlier in this file is the textbook illustration of "YES" — greedily taking the 4-coin first LOOKS locally optimal (biggest single reduction) but actually forecloses the better 3+3 solution. Recognizing this pattern of question is genuinely more valuable long-term than memorizing which specific problems are "greedy problems."

## Try this

```python
# Given a set of meetings with (start, end) times, determine the MINIMUM number of
# meeting rooms required to accommodate all of them without conflicts
# Hint: think about tracking start times and end times SEPARATELY, sorted, and using
# two pointers (file 13) to simulate rooms opening and closing over time
# Then: try to construct your OWN counterexample where a "pick the biggest item first"
# greedy strategy fails, similar in spirit to the coin change example in this file
```

Constructing a genuine counterexample from scratch (not just reading one) is honestly the single best exercise for building real judgment about when greedy is safe to reach for versus when it's a trap.

## What's next
File 15 — Dynamic Programming, genuinely the topic most self-taught learners find hardest, and directly the fix for every case where greedy provably fails — including this file's own coin change counterexample.
