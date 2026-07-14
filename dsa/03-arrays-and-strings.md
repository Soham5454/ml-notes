# 03 — Arrays & Strings

Genuinely the most-used data structures in all of DSA — most interview problems either start here or reduce down to array/string manipulation eventually.

## Python lists as arrays — what's actually happening underneath

```python
arr = [1, 2, 3, 4, 5]

arr[0]          # O(1) -- direct index access, genuinely constant time
arr.append(6)    # O(1) amortized -- Python lists over-allocate memory, so most appends don't need resizing
arr.pop()         # O(1) -- removing from the END
arr.pop(0)        # O(n) -- removing from the START, everything after has to shift left
arr.insert(0, 0)  # O(n) -- same shifting cost, inserting at the start
```

Genuinely important, commonly-missed distinction: operations at the END of a Python list are O(1), operations at the START or MIDDLE are O(n) because every subsequent element has to physically shift in memory. This directly motivates file 04's linked lists, which flip this tradeoff.

## Two pointers — a preview, fully covered in file 13

```python
def is_palindrome(s):
    left, right = 0, len(s) - 1
    while left < right:
        if s[left] != s[right]:
            return False
        left += 1
        right -= 1
    return True

print(is_palindrome("racecar"))   # True
```

`O(n)` time, `O(1)` extra space — genuinely a massive improvement over the naive `s == s[::-1]` approach in terms of demonstrating understanding (even though the naive version is fine in real code), since the two-pointer version doesn't allocate a reversed copy.

## Sliding window — a preview, fully covered in file 13

```python
def max_sum_subarray(arr, k):
    window_sum = sum(arr[:k])
    max_sum = window_sum
    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i - k]   # slide the window: add new element, drop oldest
        max_sum = max(max_sum, window_sum)
    return max_sum

print(max_sum_subarray([1, 4, 2, 10, 2, 3, 1, 0, 20], 4))   # 24
```

Genuinely important pattern: instead of recomputing the sum of every possible window from scratch (`O(n*k)`, nested loop), sliding window reuses the previous window's sum, just adjusting for what entered and left — brings it down to `O(n)`.

## Prefix sums — a genuinely high-value, underused technique

```python
def build_prefix_sums(arr):
    prefix = [0] * (len(arr) + 1)
    for i in range(len(arr)):
        prefix[i+1] = prefix[i] + arr[i]
    return prefix

def range_sum(prefix, left, right):   # inclusive range [left, right]
    return prefix[right + 1] - prefix[left]

arr = [2, 4, 6, 8, 10]
prefix = build_prefix_sums(arr)
print(range_sum(prefix, 1, 3))   # 4+6+8 = 18, computed in O(1) after O(n) preprocessing
```

Genuinely valuable when a problem asks for MANY range-sum queries on the SAME array — without prefix sums, each query costs O(n); with them, each query costs O(1) after a one-time O(n) setup. A very common "optimize this" interview follow-up.

## Kadane's Algorithm — maximum subarray sum, a genuine classic

```python
def max_subarray_sum(arr):
    max_ending_here = arr[0]
    max_so_far = arr[0]
    for num in arr[1:]:
        max_ending_here = max(num, max_ending_here + num)   # either extend, or start fresh here
        max_so_far = max(max_so_far, max_ending_here)
    return max_so_far

print(max_subarray_sum([-2, 1, -3, 4, -1, 2, 1, -5, 4]))   # 6 (subarray [4,-1,2,1])
```

Genuinely elegant, `O(n)` solution to a problem that looks like it needs `O(n^2)` (checking every possible subarray). The core insight worth internalizing: at every position, decide "does extending the previous subarray still help, or is it better to start fresh from here" — a decision made once per element, never revisiting past choices, which is precisely what keeps it linear.

## String manipulation basics — genuinely frequently tested

```python
s = "hello world"

s.split()          # ['hello', 'world']
s.upper()            # 'HELLO WORLD'
s.replace('o', '0')  # 'hell0 w0rld'
''.join(['a','b','c'])  # 'abc' -- building strings, O(n) rather than repeated += which is O(n^2) over a loop

# Genuinely important gotcha: strings in Python are IMMUTABLE
s = "hello"
# s[0] = 'H'   -- this would raise a TypeError
s = 'H' + s[1:]   # have to build a NEW string instead
```

The immutability point matters genuinely for complexity analysis — repeatedly doing `result += char` inside a loop is actually `O(n^2)` overall in the worst case (each `+=` creates a new string), whereas building a list of characters and `''.join()`-ing at the end is `O(n)`. A subtle but real interview-relevant distinction.

## Anagram checking — a genuinely common interview warm-up, multiple approaches

```python
from collections import Counter

def is_anagram_sort(s1, s2):        # O(n log n) -- sorting-based
    return sorted(s1) == sorted(s2)

def is_anagram_counter(s1, s2):     # O(n) -- hashing-based, preview of file 06
    return Counter(s1) == Counter(s2)

print(is_anagram_counter("listen", "silent"))   # True
```

Genuinely a good example of the SAME problem solvable at two different complexity levels — worth recognizing in an interview setting that jumping straight to the `O(n)` hashing solution (rather than starting with sorting) is usually the stronger answer, once hashing is covered properly in file 06.

## Try this

```python
# Given an array of integers, find if there exist two numbers that sum to a given target
# 1. First write the naive O(n^2) nested-loop solution
# 2. Then rewrite it using a hash set for O(n) time (preview of file 06's technique)
# 3. Compare: for an array of 100,000 elements, roughly estimate how many operations
#    each approach would need (recap file 01's growth-rate comparison)
```

This exact problem ("Two Sum") is genuinely one of the most commonly asked interview questions of all time — solving it both ways, and being able to explain WHY the hash-based version is better, is a foundational skill this whole roadmap builds toward.

## What's next
File 04 — Linked Lists, a data structure that flips arrays' O(n) insert/delete-at-start cost into O(1), at the cost of losing O(1) random access.
