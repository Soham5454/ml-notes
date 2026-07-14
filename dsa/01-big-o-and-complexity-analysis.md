# 01 — Big O & Complexity Analysis

Starting the DSA phase now, in Python since it's already my main language across all the ML/DL notes. This file is genuinely the language every other DSA file will speak — "how fast" and "how much memory" a solution uses.

## Why complexity analysis matters at all

Two solutions can both give the CORRECT answer but behave completely differently as input size grows. A solution that takes 1 second for 100 items might take 3 hours for 100,000 items if it's inefficient — complexity analysis predicts this BEFORE running the code, just by looking at its structure.

## Big O — worst-case growth rate

Big O describes how an algorithm's runtime (or memory) grows as input size `n` grows, ignoring constants and focusing on the DOMINANT term.

```python
def find_max(arr):          # O(n) - one pass through the array
    max_val = arr[0]
    for num in arr:          # loop runs n times
        if num > max_val:
            max_val = num
    return max_val
```

## Common complexity classes — genuinely worth memorizing in order

```
O(1)        - constant       - dict lookup, array index access
O(log n)    - logarithmic    - binary search
O(n)        - linear         - single loop through data
O(n log n)  - linearithmic   - efficient sorting (merge sort, quick sort avg case)
O(n^2)      - quadratic      - nested loops, bubble sort
O(2^n)      - exponential    - naive recursive fibonacci, subset generation
O(n!)       - factorial      - generating all permutations
```

```python
# O(1) - constant time, regardless of dict size
d = {'a': 1, 'b': 2}
value = d['a']          # instant lookup, doesn't matter if dict has 10 or 10 million keys

# O(n) - linear
def linear_search(arr, target):
    for i, val in enumerate(arr):
        if val == target:
            return i
    return -1

# O(n^2) - quadratic, nested loop over the same data
def has_duplicate_naive(arr):
    for i in range(len(arr)):
        for j in range(i+1, len(arr)):
            if arr[i] == arr[j]:
                return True
    return False

# O(log n) - logarithmic, cuts the problem in half each step (full treatment in file 12)
def binary_search(arr, target):
    low, high = 0, len(arr) - 1
    while low <= high:
        mid = (low + high) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1
```

Genuinely the most important intuition to build: `O(log n)` is DRAMATICALLY faster than `O(n)` at scale — binary search on 1 billion sorted items takes ~30 comparisons; linear search takes up to 1 billion. This exact gap is why sorted data structures and binary search show up constantly in interview-optimal solutions.

## Why constants and lower-order terms get dropped

```python
# This is technically O(2n + 5), but Big O notation drops constants and lower-order terms
def process(arr):
    total = 0
    for num in arr:              # n iterations
        total += num
    for num in arr:              # another n iterations
        total += num * 2
    return total + 5              # constant work
# Big O: O(n), not O(2n+5) -- as n grows huge, the "2" and "+5" become irrelevant to the GROWTH RATE
```

Big O cares about growth trend, not exact operation counts — genuinely the point is answering "how does this scale," not "exactly how many operations."

## Space complexity — the memory side of the same coin

```python
def sum_array(arr):              # O(1) space -- uses a fixed amount of extra memory regardless of n
    total = 0
    for num in arr:
        total += num
    return total

def duplicate_array(arr):        # O(n) space -- creates a new list proportional to input size
    return arr + arr

def fibonacci_recursive(n):      # O(n) space -- due to the call stack depth (recap file 02)
    if n <= 1:
        return n
    return fibonacci_recursive(n-1) + fibonacci_recursive(n-2)
```

Genuinely a real, common interview follow-up: "can you do this in O(1) extra space" — meaning solve it WITHOUT creating new data structures proportional to input size, often by modifying data in-place instead.

## Best, worst, and average case — why Big O usually means WORST case

```python
def linear_search(arr, target):
    for i, val in enumerate(arr):
        if val == target:          # best case: O(1) if target is first element
            return i
    return -1                       # worst case: O(n) if target is last element or absent
```

Big O notation, used alone, conventionally refers to the WORST case unless stated otherwise — genuinely the safer assumption for interview/production purposes, since it guarantees an upper bound no matter how "unlucky" the input is.

## Comparing growth rates concretely — the exercise that makes this real

```python
n = 1000
import math

print(f"O(log n): {math.log2(n):.0f}")      # ~10
print(f"O(n): {n}")                          # 1000
print(f"O(n log n): {n * math.log2(n):.0f}")  # ~9966
print(f"O(n^2): {n**2}")                      # 1,000,000
print(f"O(2^n): far too large to even compute for n=1000")
```

Seeing these numbers side by side for a realistic input size is genuinely the fastest way to internalize WHY interviewers care so much about moving a solution from O(n²) to O(n log n) or O(n) — at real scale, the difference isn't "a bit slower," it's the difference between milliseconds and literally not finishing in a human lifetime.

## Try this

```python
# For each function below, determine its Big O time complexity BEFORE running any code,
# just by reading the structure -- then explain WHY out loud/in writing

def mystery1(arr):
    for i in range(len(arr)):
        print(arr[i])

def mystery2(arr):
    for i in range(len(arr)):
        for j in range(len(arr)):
            print(arr[i], arr[j])

def mystery3(arr):
    if len(arr) == 0:
        return
    print(arr[0])
    mystery3(arr[1:])
```

Being able to look at raw code and immediately state its Big O — without running it, without counting exact operations — is genuinely the core skill this whole file exists to build. Every algorithm covered from here on (files 02-19) will get evaluated using exactly this lens.

## What's next
File 02 — Recursion & Backtracking, the problem-solving pattern that underlies tree/graph traversal, dynamic programming, and a huge fraction of "hard" interview problems.
