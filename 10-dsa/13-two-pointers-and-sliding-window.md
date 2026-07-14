# 13 — Two Pointers & Sliding Window

Recap file 03's palindrome and max-sum-subarray previews — this file gives both techniques their full, proper treatment, since they're genuinely among the highest-frequency patterns across real interviews.

## Two Pointers — opposite ends, converging inward

```python
def two_sum_sorted(arr, target):          # arr is SORTED
    left, right = 0, len(arr) - 1
    while left < right:
        current_sum = arr[left] + arr[right]
        if current_sum == target:
            return [left, right]
        elif current_sum < target:
            left += 1                        # need a BIGGER sum, move left pointer up
        else:
            right -= 1                        # need a SMALLER sum, move right pointer down
    return []

print(two_sum_sorted([2, 7, 11, 15], 9))   # [0, 1]
```

`O(n)` time, `O(1)` space — genuinely a real improvement over file 06's hash-based Two Sum (`O(n)` space) SPECIFICALLY when the array is already sorted, since the sortedness itself provides the information needed to know which direction to move.

## Two pointers — same direction (fast/slow), a genuinely different sub-pattern

```python
def remove_duplicates(arr):          # arr is sorted, remove duplicates IN PLACE
    if not arr:
        return 0
    slow = 0
    for fast in range(1, len(arr)):
        if arr[fast] != arr[slow]:
            slow += 1
            arr[slow] = arr[fast]
    return slow + 1                    # new length of the deduplicated portion

arr = [1, 1, 2, 2, 3, 4, 4]
new_length = remove_duplicates(arr)
print(arr[:new_length])   # [1, 2, 3, 4]
```

Recap file 04's fast/slow pointer note for cycle detection — this is genuinely the SAME two-speed-pointer idea, repurposed: `slow` marks the boundary of "processed, deduplicated" data, `fast` scans ahead looking for the next genuinely new value. `O(n)` time, `O(1)` extra space — a real, common "modify array in place" requirement in interviews.

## Sliding Window — fixed size, recap and full context

```python
def max_sum_subarray(arr, k):          # recap file 03, now with full explanation
    window_sum = sum(arr[:k])
    max_sum = window_sum
    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i - k]   # ADD the new element entering, SUBTRACT the one leaving
        max_sum = max(max_sum, window_sum)
    return max_sum
```

## Sliding Window — variable size, the genuinely more common interview variant

```python
def smallest_subarray_with_sum(arr, target):
    left = 0
    current_sum = 0
    min_length = float('inf')

    for right in range(len(arr)):
        current_sum += arr[right]                # expand window by including arr[right]

        while current_sum >= target:                # shrink window from the left while condition holds
            min_length = min(min_length, right - left + 1)
            current_sum -= arr[left]
            left += 1

    return min_length if min_length != float('inf') else 0

print(smallest_subarray_with_sum([2, 1, 5, 2, 3, 2], 7))   # 2 (subarray [5,2])
```

Genuinely the core template worth memorizing: EXPAND the window by moving `right` forward, and whenever some condition is satisfied (or violated, depending on the problem), SHRINK from `left` until it no longer holds — each element gets added and removed from the window AT MOST once across the whole run, which is precisely why this achieves `O(n)` despite looking like it might be doing nested-loop work.

## Longest substring without repeating characters — a genuinely classic variable-window problem

```python
def longest_unique_substring(s):
    seen = {}          # recap file 06's hashing -- char -> most recent index
    left = 0
    max_length = 0

    for right, char in enumerate(s):
        if char in seen and seen[char] >= left:
            left = seen[char] + 1        # jump left pointer PAST the previous occurrence
        seen[char] = right
        max_length = max(max_length, right - left + 1)

    return max_length

print(longest_unique_substring("abcabcbb"))   # 3 ("abc")
```

Genuinely a great example of two files combining: file 06's hash map for O(1) lookups of "have I seen this character, and where," fused with the expand/shrink sliding window template — a very common real interview question precisely because it tests both patterns simultaneously.

## Container With Most Water — two pointers on a genuinely different kind of problem

```python
def max_water_container(heights):
    left, right = 0, len(heights) - 1
    max_area = 0

    while left < right:
        width = right - left
        height = min(heights[left], heights[right])
        max_area = max(max_area, width * height)

        if heights[left] < heights[right]:      # move the SHORTER side -- genuinely the key insight
            left += 1
        else:
            right -= 1

    return max_area

print(max_water_container([1, 8, 6, 2, 5, 4, 8, 3, 7]))   # 49
```

Genuinely worth understanding WHY moving the shorter side is correct, not just memorizing it: the container's area is limited by the SHORTER of the two walls — keeping the taller wall and moving the shorter one is the only way that COULD possibly find a larger area (moving the taller wall inward can only ever decrease or maintain width while the limiting height stays the same or gets worse). This reasoning-through-why is genuinely the difference between memorizing solutions and being able to derive new ones under interview pressure.

## Sliding window maximum — combining with file 05's monotonic deque

```python
from collections import deque

def sliding_window_max(arr, k):
    dq = deque()          # stores INDICES, maintains DECREASING values front to back
    result = []

    for i in range(len(arr)):
        while dq and arr[dq[-1]] < arr[i]:
            dq.pop()                        # remove smaller elements -- they can never be the max again
        dq.append(i)

        if dq[0] <= i - k:                    # remove the front if it's fallen out of the window
            dq.popleft()

        if i >= k - 1:
            result.append(arr[dq[0]])           # front of deque is always the current window's max

    return result

print(sliding_window_max([1,3,-1,-3,5,3,6,7], 3))   # [3, 3, 5, 5, 6, 7]
```

Recap file 05's monotonic stack (next greater element) and deque notes — this is genuinely the SAME "maintain a monotonic structure, evict irrelevant elements" idea, now combined with a sliding window. `O(n)` overall, since each index is added/removed from the deque at most once.

## Try this

```python
# Given a string, find the length of the longest substring that contains AT MOST k
# distinct characters (a generalization of the "longest unique substring" problem above)
# Then solve: given an array of integers, find the number of subarrays whose sum
# EQUALS a target exactly (hint: this one is genuinely NOT solvable with the simple
# expand/shrink window alone if negative numbers are allowed -- think about combining
# it with file 06's hashing/prefix-sum idea instead)
```

That second problem is a genuinely good test of pattern RECOGNITION — knowing when sliding window applies cleanly (monotonic behavior, all-positive sums) versus when a different combination of techniques (prefix sums + hashing) is actually required instead.

## What's next
File 14 — Greedy Algorithms, a fundamentally different problem-solving philosophy: making the locally-best choice at each step and trusting (with proof, when it's genuinely valid) that it leads to a globally optimal solution.
