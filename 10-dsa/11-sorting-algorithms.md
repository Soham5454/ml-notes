# 11 — Sorting Algorithms

First file of the pure Algorithms section. Genuinely foundational — sorting shows up as a SUBROUTINE inside countless other algorithms (binary search needs sorted data, greedy algorithms often sort first, merge-based techniques reappear in file 16).

## Why multiple sorting algorithms exist at all

Genuinely no single "best" sorting algorithm — different ones win on different criteria: time complexity, space complexity, stability (whether equal elements keep their relative order), and behavior on nearly-sorted or small inputs. Python's built-in `sorted()`/`.sort()` already handles all of this well (uses Timsort, a hybrid), but understanding the classics is genuinely what interviews test.

## Bubble Sort — the simplest, and genuinely the one to understand THEN move past

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(n - i - 1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = True
        if not swapped:      # early exit if already sorted -- genuinely worth including
            break
    return arr

print(bubble_sort([5, 2, 8, 1, 9]))   # [1, 2, 5, 8, 9]
```

`O(n²)` time, `O(1)` space. Genuinely rarely used in practice — included specifically because it's the clearest illustration of "compare adjacent, swap if needed, repeat" before moving to smarter approaches.

## Merge Sort — divide and conquer, genuinely important

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])       # recursively sort left half
    right = merge_sort(arr[mid:])       # recursively sort right half

    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result

print(merge_sort([5, 2, 8, 1, 9, 3]))   # [1, 2, 3, 5, 8, 9]
```

Recap file 02's recursion — merge sort genuinely IS the divide-and-conquer pattern: break the problem in half, solve each half recursively, then COMBINE the solved halves (the `merge` step). `O(n log n)` time — the `log n` comes from how many times the array can be halved (recap file 01), and the `n` comes from the merge step touching every element at each level. `O(n)` space — the real cost compared to some alternatives, since merging needs extra arrays.

## Quick Sort — genuinely the most commonly asked-about sort in interviews

```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr

    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]

    return quick_sort(left) + middle + quick_sort(right)

print(quick_sort([5, 2, 8, 1, 9, 3]))   # [1, 2, 3, 5, 8, 9]
```

Core idea: pick a **pivot**, partition the array into "smaller than pivot" and "larger than pivot," then recursively sort each partition. `O(n log n)` average case, but genuinely `O(n²)` WORST case (e.g. if the pivot is always the smallest or largest element, like on an already-sorted array with a naive pivot choice) — this worst-case risk is precisely why "choosing a good pivot" (random, median-of-three) matters in real implementations.

```python
# An in-place version, closer to what's actually asked for in interviews (O(log n) space via recursion, not O(n))
def quick_sort_inplace(arr, low=0, high=None):
    if high is None:
        high = len(arr) - 1
    if low < high:
        pivot_index = partition(arr, low, high)
        quick_sort_inplace(arr, low, pivot_index - 1)
        quick_sort_inplace(arr, pivot_index + 1, high)
    return arr

def partition(arr, low, high):
    pivot = arr[high]
    i = low - 1
    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i+1], arr[high] = arr[high], arr[i+1]
    return i + 1
```

## Merge Sort vs Quick Sort — the honest comparison

| | Merge Sort | Quick Sort |
|---|---|---|
| Average time | O(n log n) | O(n log n) |
| Worst time | O(n log n) — GUARANTEED | O(n²) — possible with bad pivots |
| Space | O(n) | O(log n) average (in-place version) |
| Stability | Stable (equal elements keep order) | Not stable by default |
| Practical use | Good when worst-case guarantee matters, external sorting (data too big for memory) | Generally faster in practice due to better cache locality, used in many language standard libraries |

## Counting Sort — a genuinely different, non-comparison-based approach

```python
def counting_sort(arr):
    if not arr:
        return arr
    max_val = max(arr)
    counts = [0] * (max_val + 1)

    for num in arr:
        counts[num] += 1

    result = []
    for num, count in enumerate(counts):
        result.extend([num] * count)

    return result

print(counting_sort([4, 2, 2, 8, 3, 3, 1]))   # [1, 2, 2, 3, 3, 4, 8]
```

Genuinely `O(n + k)` time, where `k` is the range of input values — can beat `O(n log n)` comparison sorts when `k` is small relative to `n`. Important honest limitation: only works for integers (or discretely-bucketable data) within a KNOWN, reasonably small range — completely impractical for sorting, say, arbitrary floating point numbers or a huge value range.

## Python's built-in sort — what to actually use in real code

```python
arr = [5, 2, 8, 1, 9]
arr.sort()                    # in-place, O(n log n), stable (Timsort)
sorted_arr = sorted(arr)       # returns a new list

# Custom sort key -- genuinely useful and common
words = ["banana", "kiwi", "apple"]
words.sort(key=len)             # sort by string length instead of alphabetically
print(words)                     # ['kiwi', 'apple', 'banana']

people = [("Alice", 30), ("Bob", 25), ("Carol", 35)]
people.sort(key=lambda p: p[1])   # sort by age (second tuple element)
```

Genuinely important practical point: in REAL code (not interview whiteboard settings), always use Python's built-in `sorted()`/`.sort()` — it's a highly optimized Timsort implementation, genuinely faster and more robust than any hand-rolled sort. The manual implementations above exist for interview/understanding purposes, not because they should replace the built-in in production.

## Try this

```python
# Given an array of objects representing students with (name, grade, age),
# sort them by grade DESCENDING, and for students with the SAME grade,
# sort by age ASCENDING as a tiebreaker
# Hint: Python's sort is stable, and supports tuple keys -- e.g. key=lambda s: (-s[1], s[2])
```

Multi-criteria sorting with tiebreakers is genuinely a common real interview/practical task — the negative-value trick for descending order on one field while ascending on another is worth having in the toolkit.

## What's next
File 12 — Searching & Binary Search, extending file 01's brief binary search mention into the full pattern, including its many less-obvious variants (search in rotated arrays, finding boundaries) that show up constantly in interviews.
