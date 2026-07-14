# 19 — Problem-Solving Framework

The final DSA file — not a new technique, but the actual STEP-BY-STEP process for turning "here's a problem I've never seen" into a working, well-communicated solution under real interview conditions.

## Step 1 — Understand the problem completely, BEFORE thinking about solutions

Genuinely the most commonly skipped step, and the source of a huge fraction of avoidable mistakes. Actively ask:
- What EXACTLY is the input? (type, size, constraints — genuinely read the constraints, they hint at the expected complexity)
- What EXACTLY is the output?
- What are the EDGE CASES? (empty input, single element, all duplicates, negative numbers, very large input)
- Restate the problem in my own words, OUT LOUD if in an interview — this alone catches a genuinely surprising number of misunderstandings before any code gets written.

```python
# Example: "Given an array, find the maximum subarray sum"
# Clarifying questions worth asking:
# - Can the array be empty? Can it contain negative numbers? (recap file 03's Kadane's algorithm
#   specifically handles negatives -- this constraint genuinely changes the approach)
# - Does "subarray" mean contiguous, or could it mean any subset? (huge difference!)
```

## Step 2 — Work through a small example BY HAND

Genuinely before writing any code, trace through a small, concrete example manually. This surfaces the actual RECURRENCE or PATTERN needed, and catches misunderstandings from Step 1 that weren't yet obvious.

```
Example: [−2, 1, −3, 4, −1, 2, 1, −5, 4]
Manually tracking "best subarray ending at each position" (recap file 03's Kadane's algorithm)
reveals the actual recurrence needed BEFORE any code gets written.
```

## Step 3 — Identify the pattern (recap file 18)

Given the constraints and the manual trace from steps 1-2, which of file 18's ~15 patterns does this resemble? Genuinely say this OUT LOUD in an interview — "this looks like a sliding window problem because we need a contiguous subarray and the constraint changes as the window grows" demonstrates the reasoning process, which matters as much as the final answer to most interviewers.

## Step 4 — Start with the BRUTE FORCE solution, explicitly

Genuinely resist the urge to jump straight to the optimal solution. State the brute force approach and its complexity FIRST:

```python
# Brute force for max subarray sum: check every possible subarray -- O(n^2) or O(n^3)
def brute_force_max_subarray(arr):
    max_sum = float('-inf')
    for i in range(len(arr)):
        current_sum = 0
        for j in range(i, len(arr)):
            current_sum += arr[j]
            max_sum = max(max_sum, current_sum)
    return max_sum
```

This genuinely serves two real purposes: it's a safety net (a working, if slow, solution beats no solution), and it gives a concrete BASELINE to explain the improvement against when presenting the optimized version.

## Step 5 — Optimize, explaining the improvement explicitly

```python
# Optimized: Kadane's algorithm, O(n) -- recap file 03
def optimized_max_subarray(arr):
    max_ending_here = max_so_far = arr[0]
    for num in arr[1:]:
        max_ending_here = max(num, max_ending_here + num)
        max_so_far = max(max_so_far, max_ending_here)
    return max_so_far
```

Genuinely worth explicitly stating WHAT changed and WHY it's faster: "instead of recomputing every possible subarray sum from scratch, I track the best sum ENDING at each position, reusing the previous result — this avoids redundant work, bringing it from O(n²) to O(n)."

## Step 6 — Code it cleanly, narrating while writing (in an interview context)

Genuinely good habits, real interview-relevant:
- Use clear variable names (`left`/`right` for two pointers, not `i`/`j` — recap file 13).
- Handle edge cases explicitly, don't just hope they work.
- Narrate the reasoning WHILE coding, not silently — interviewers are evaluating the THINKING process, not just the final output.

## Step 7 — Test with the manual example from Step 2, then edge cases

```python
# Test with the original hand-traced example
print(optimized_max_subarray([-2, 1, -3, 4, -1, 2, 1, -5, 4]))   # 6, matches manual trace

# Test edge cases explicitly
print(optimized_max_subarray([-1]))              # -1, single negative element
print(optimized_max_subarray([-3, -1, -2]))       # -1, all negative -- still correctly finds the LEAST negative
```

Genuinely a real, common mistake: forgetting to verify against ALL-NEGATIVE input specifically — a naive "start max_sum at 0" implementation would incorrectly return 0 instead of the correct answer (the least-negative single element), a bug that only shows up on this specific edge case.

## Step 8 — State the final complexity, both time and space

Genuinely always finish by explicitly stating: "This solution runs in O(n) time and O(1) space." Interviewers expect this stated clearly, not left implicit.

## The honest checklist to run through for EVERY problem

```
[ ] Restated the problem in my own words
[ ] Asked about edge cases / constraints
[ ] Traced a small example by hand
[ ] Identified the likely pattern (file 18)
[ ] Started with brute force, stated its complexity
[ ] Optimized, explained WHY it's better
[ ] Coded cleanly with clear variable names
[ ] Tested against the hand-traced example
[ ] Tested edge cases (empty, single element, all same, negatives, etc.)
[ ] Stated final time AND space complexity
```

## What this framework is genuinely for

Not every step needs equal time on every problem — an easy problem might blur steps 4-5 together, a hard problem might spend most of the time on step 1-3. But genuinely having this checklist as a mental default prevents the two most common interview failure modes: jumping to code too fast (skipping 1-3) and finishing code without verifying it (skipping 7-9).

## Try this — the actual capstone exercise for the whole DSA folder

```python
# Pick 3 problems, one EASY, one MEDIUM, one HARD, from any interview problem source
# For EACH one, run through all 9 checklist steps EXPLICITLY, writing out each step
# (not just thinking it internally) -- including the brute force attempt even if the
# optimal solution seems obvious immediately
# This is genuinely the exercise that builds the actual interview MUSCLE, not just
# theoretical knowledge -- the difference between "knowing DSA" and "performing well
# under real interview pressure" comes down almost entirely to having run this process
# enough times that it becomes automatic
```

## DSA folder — fully complete

Files 01-19 covered: complexity analysis, recursion/backtracking, every core data structure (arrays, linked lists, stacks/queues, hashing, trees, heaps, graphs, tries), every core algorithm family (sorting, searching, two pointers/sliding window, greedy, DP, graph algorithms, bit manipulation), pattern recognition across ~15 recurring interview patterns, and this final problem-solving framework tying it all into an actual repeatable process.

## Roadmap position, updated
Python Basics → NumPy → pandas → Matplotlib → Seaborn → scikit-learn → XGBoost → PyTorch → Math Foundations → **DSA (done)** → SQL → Classical NLP → Hugging Face → OpenCV → Flask → MLOps → Cloud → Frontier GenAI → ML System Design → Interview Prep
