# 12 — Searching & Binary Search

Recap file 01's brief binary search introduction — this file goes deep, because binary search is genuinely one of the highest-value patterns in all of interview DSA, with far more variants than most people initially realize.

## The core template, precisely

```python
def binary_search(arr, target):
    low, high = 0, len(arr) - 1
    while low <= high:
        mid = low + (high - low) // 2      # avoids potential overflow, genuinely good habit
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1
```

`O(log n)` time, `O(1)` space — recap file 01's growth comparison, this is precisely why it's so much faster than linear search at scale. Requirement genuinely non-negotiable: the array MUST be sorted for standard binary search to work at all.

## The mental model — genuinely worth internalizing precisely

At every step, binary search asks ONE question ("is the target bigger or smaller than the middle element") and ELIMINATES HALF the remaining search space based on the answer. This "eliminate half" idea is the actual generalizable pattern — many binary search problems don't look like searching a sorted array at all, but share this exact "can I eliminate half the possibilities based on one check" structure.

## Finding the first/last occurrence — a genuinely common variant

```python
def find_first_occurrence(arr, target):
    low, high = 0, len(arr) - 1
    result = -1
    while low <= high:
        mid = low + (high - low) // 2
        if arr[mid] == target:
            result = mid
            high = mid - 1          # keep searching LEFT for an earlier occurrence
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return result

print(find_first_occurrence([1, 2, 2, 2, 3, 4], 2))   # 1 (index of the FIRST 2)
```

Genuinely important pattern worth internalizing: even after FINDING a match, don't stop immediately — keep narrowing in the direction needed (left for "first occurrence," right for "last occurrence"), recording the best answer found so far.

## Binary search on the ANSWER — the genuinely powerful, less obvious pattern

Many problems don't hand you a sorted array at all — instead, the ANSWER itself lives on a sorted/monotonic range, and binary search can be applied directly to that range of possible answers.

```python
# Example: "minimum number of days to make m bouquets" style problem, simplified here to
# a genuinely common variant: find the smallest divisor such that the sum of
# ceil(x/divisor) for all x in arr is <= threshold

import math

def smallest_divisor(arr, threshold):
    def compute_sum(divisor):
        return sum(math.ceil(x / divisor) for x in arr)

    low, high = 1, max(arr)
    while low < high:
        mid = low + (high - low) // 2
        if compute_sum(mid) <= threshold:
            high = mid          # mid MIGHT be small enough -- try smaller
        else:
            low = mid + 1        # mid is too small a divisor (sum too big) -- need bigger
    return low

print(smallest_divisor([1, 2, 5, 9], 6))   # 5
```

Genuinely the key recognition skill: whenever a problem asks for the "minimum X such that condition holds" or "maximum X such that condition holds," and increasing X monotonically makes the condition easier (or harder) to satisfy, binary search over the RANGE OF POSSIBLE ANSWERS applies — even though nothing about the original input array needed to be sorted.

## Search in a rotated sorted array — a genuinely classic, trickier variant

```python
def search_rotated(arr, target):
    low, high = 0, len(arr) - 1
    while low <= high:
        mid = low + (high - low) // 2
        if arr[mid] == target:
            return mid

        if arr[low] <= arr[mid]:            # left half is properly sorted
            if arr[low] <= target < arr[mid]:
                high = mid - 1
            else:
                low = mid + 1
        else:                                  # right half is properly sorted instead
            if arr[mid] < target <= arr[high]:
                low = mid + 1
            else:
                high = mid - 1
    return -1

print(search_rotated([4, 5, 6, 7, 0, 1, 2], 0))   # 4
```

Genuinely the key insight: even though the WHOLE array isn't sorted (it's been "rotated" at some pivot point), at least ONE HALF (left or right of `mid`) is always properly sorted at any given step — checking WHICH half is sorted, then checking whether the target falls within that sorted half's range, lets binary search's elimination logic still apply.

## `bisect` — Python's built-in binary search module

```python
import bisect

arr = [1, 3, 3, 5, 7, 9]

bisect.bisect_left(arr, 3)     # 1 -- leftmost position to insert 3, keeping sorted order
bisect.bisect_right(arr, 3)    # 3 -- rightmost position to insert 3

bisect.insort(arr, 4)           # inserts 4 into arr, keeping it sorted -- O(n) due to shifting, but O(log n) to FIND the spot
print(arr)                        # [1, 3, 3, 4, 5, 7, 9]
```

Genuinely worth knowing this module exists — `bisect_left`/`bisect_right` directly implement the "first/last occurrence" style logic from earlier in this file, and are the standard, correct tool to reach for in actual Python code rather than hand-rolling binary search every time.

## Peak element — binary search without a fully sorted array at all

```python
def find_peak_element(arr):
    low, high = 0, len(arr) - 1
    while low < high:
        mid = low + (high - low) // 2
        if arr[mid] > arr[mid + 1]:
            high = mid           # peak is at mid or to the LEFT
        else:
            low = mid + 1          # peak is to the RIGHT
    return low

print(find_peak_element([1, 2, 3, 1]))   # 2 (index of value 3, a local peak)
```

Genuinely another good example of "binary search doesn't require full sortedness" — it only requires that at every step, ONE comparison can correctly eliminate half the remaining candidates, which holds here because comparing `arr[mid]` to its neighbor always points toward SOME peak.

## Try this

```python
# Given a sorted array (possibly with duplicates) and a target, find BOTH the first and
# last index of the target in one combined solution (reuse the find_first_occurrence
# pattern, adapt it for "last occurrence" by flipping the narrowing direction)
# Then solve: given a sorted 2D matrix (each row sorted, first element of each row greater
# than the last element of the previous row), search for a target treating the whole
# matrix as one flattened sorted array, using index math instead of actually flattening it
```

The 2D matrix search is genuinely a good test of whether the "binary search is about eliminating half the space" mental model has generalized beyond simple 1D arrays — converting a flat index back into `(row, col)` via `divmod` is the key trick.

## What's next
File 13 — Two Pointers & Sliding Window, formalizing the techniques previewed in file 03 into their full range of variants, genuinely one of the highest-frequency pattern categories in real interviews.
