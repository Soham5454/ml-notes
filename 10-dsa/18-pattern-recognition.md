# 18 — Pattern Recognition (The ~15 Interview Patterns)

Genuinely the highest-leverage file in this whole DSA folder for actual interview performance. Files 01-17 covered individual techniques in isolation — this file is the map connecting "what does this problem LOOK like" to "which technique(s) from files 01-17 actually apply."

## Why pattern recognition matters more than memorizing solutions

There are genuinely thousands of possible interview questions, but they overwhelmingly reduce to a much smaller set of recurring PATTERNS. Recognizing "oh, this is a sliding window problem" (even if the specific problem has never been seen before) is a transferable skill — memorizing "the answer to problem #47 is this specific code" is not.

## Pattern 1 — Two Pointers
**Recognize when:** sorted array/string, looking for a pair/triplet, palindrome checks, or "opposite ends converging."
**Recap:** file 13. **Example tell:** "find two numbers that sum to X in a SORTED array."

## Pattern 2 — Sliding Window
**Recognize when:** contiguous subarray/substring, "longest/shortest/maximum" phrasing, constraint that changes as the window grows/shrinks.
**Recap:** file 13. **Example tell:** "longest substring with at most K distinct characters."

## Pattern 3 — Fast & Slow Pointers
**Recognize when:** linked lists, cycle detection, finding a middle element, "meeting point" phrasing.
**Recap:** file 04. **Example tell:** "detect a cycle in a linked list."

## Pattern 4 — Merge Intervals
**Recognize when:** array of (start, end) ranges, overlapping/scheduling phrasing.
```python
def merge_intervals(intervals):
    intervals.sort(key=lambda x: x[0])          # recap file 11's custom sort key
    merged = [intervals[0]]
    for current in intervals[1:]:
        if current[0] <= merged[-1][1]:            # overlaps with the last merged interval
            merged[-1][1] = max(merged[-1][1], current[1])
        else:
            merged.append(current)
    return merged
```
**Example tell:** "merge overlapping meeting times," directly related to file 14's activity selection.

## Pattern 5 — Cyclic Sort
**Recognize when:** array containing numbers in a known range (like 1 to n), "find the missing/duplicate number" phrasing.
```python
def find_missing_number(nums):          # nums contains n distinct numbers from 0 to n
    i, n = 0, len(nums)
    while i < n:
        correct_index = nums[i]
        if correct_index < n and nums[i] != nums[correct_index]:
            nums[i], nums[correct_index] = nums[correct_index], nums[i]
        else:
            i += 1
    for i in range(n):
        if nums[i] != i:
            return i
    return n
```
**Example tell:** "find all missing numbers from 1 to n" — genuinely `O(n)` time, `O(1)` space by placing each number at its "correct" index first.

## Pattern 6 — In-place Linked List Reversal
**Recognize when:** "reverse a linked list," possibly in groups/sublists.
**Recap:** file 04's reverse_linked_list. **Example tell:** "reverse every K nodes in a linked list."

## Pattern 7 — BFS (Tree/Graph)
**Recognize when:** "level order," "shortest path" (unweighted), "minimum steps," "nearest."
**Recap:** files 07 and 09. **Example tell:** "minimum number of steps to reach the end of a maze."

## Pattern 8 — DFS (Tree/Graph)
**Recognize when:** "all paths," "explore every possibility," tree properties (height, sum, validity), connected components.
**Recap:** files 07 and 09. **Example tell:** "find all root-to-leaf paths summing to a target."

## Pattern 9 — Two Heaps
**Recognize when:** need BOTH the smallest AND largest half simultaneously, "median" phrasing, "schedule."
**Recap:** file 08's median-from-stream "try this." **Example tell:** "find the median of a running data stream."

## Pattern 10 — Subsets / Backtracking
**Recognize when:** "all combinations," "all permutations," "all possible ways," constraint satisfaction (Sudoku, N-Queens).
**Recap:** file 02. **Example tell:** "generate all valid combinations of balanced parentheses."

## Pattern 11 — Modified Binary Search
**Recognize when:** sorted (or partially sorted/rotated) array, "find the minimum X such that," search space that can be halved even without an obviously sorted array.
**Recap:** file 12. **Example tell:** "find the minimum eating speed to finish all bananas in H hours."

## Pattern 12 — Top K Elements
**Recognize when:** "K largest/smallest/most frequent," don't need full sorting, just the top K.
**Recap:** file 08's kth_largest. **Example tell:** "K most frequent words in a document."

## Pattern 13 — K-way Merge
**Recognize when:** multiple SORTED arrays/lists need combining.
**Recap:** file 08's merge_k_sorted_lists. **Example tell:** "merge K sorted linked lists."

## Pattern 14 — Dynamic Programming (0/1 Knapsack family, sequence family)
**Recognize when:** "maximum/minimum," "number of ways to," choices with constraints, optimal substructure smell, greedy provably fails (recap file 14's coin-change counterexample).
**Recap:** file 15. **Example tell:** "maximum value of items fitting in a knapsack," "longest common subsequence."

## Pattern 15 — Topological Sort
**Recognize when:** dependencies, prerequisites, "ordering," directed graph with a "must happen before" relationship.
**Recap:** file 16. **Example tell:** "course schedule with prerequisites."

## Bonus Pattern — Trie
**Recognize when:** prefix matching, autocomplete, word search in a grid, dictionary-based string problems.
**Recap:** file 10. **Example tell:** "implement autocomplete" or "word search II."

## The genuinely practical skill — matching problem PHRASING to pattern

```
"longest / shortest CONTIGUOUS subarray/substring"     -> Sliding Window
"all possible / all combinations / all ways"            -> Backtracking
"K largest / K most frequent / top K"                    -> Heap (Top K)
"minimum steps / shortest path (unweighted)"              -> BFS
"sorted array, find a pair/triplet"                        -> Two Pointers
"cycle in a linked list" / "middle of a linked list"         -> Fast & Slow Pointers
"maximum/minimum value achievable, given constraints"          -> Dynamic Programming
"schedule / prerequisite / must happen before"                   -> Topological Sort
"overlapping ranges / meeting rooms"                                -> Merge Intervals
"find missing/duplicate in range 1 to n"                              -> Cyclic Sort
```

Genuinely worth building this recognition as an actual HABIT — reading a new problem statement and immediately asking "which of these ~15 patterns does the phrasing point toward" before writing a single line of code is precisely the skill that separates "solved it because I'd seen this exact problem before" from "solved it because I recognized the underlying shape."

## Try this

```python
# Take 5 problems from LeetCode's "Top Interview 150" list (or similar) that haven't been
# solved before -- for EACH one, before writing any code, write down:
# 1. Which pattern(s) from this file it looks like
# 2. Which specific phrase in the problem statement gave that away
# 3. Which file(s) 01-17 the actual technique needed comes from
# THEN attempt the solution
```

This exercise, done honestly (writing the pattern guess down BEFORE coding, not after), is genuinely the single best way to build real pattern-recognition speed for actual interview conditions, where the clock is running and there's no time to try five different approaches randomly.

## What's next
File 19 — Problem-Solving Framework, the final file: not another pattern, but the actual STEP-BY-STEP process to follow once a problem and its likely pattern have been identified, all the way through to a working, communicated solution.
