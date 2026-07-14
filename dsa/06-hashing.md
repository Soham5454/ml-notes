# 06 — Hashing

Recap file 03's Two Sum preview — this is the file that properly explains the hash-based O(n) solution, and genuinely the single highest-leverage technique in all of interview DSA for turning brute-force O(n²) into O(n).

## What a hash function does

A hash function takes an input (of any size) and produces a fixed-size number (the hash) — used as an INDEX into an underlying array, giving average O(1) lookup/insert/delete, regardless of how many items are stored.

```python
d = {}
d['apple'] = 5
d['banana'] = 3

print(d['apple'])   # O(1) average case -- doesn't matter if dict has 10 or 10 million keys
print('apple' in d)  # O(1) average case membership check
```

This is genuinely THE reason Python's `dict` and `set` feel "magically fast" compared to searching through a list — a list membership check (`x in my_list`) is O(n), while `x in my_set` is O(1) average case, because of hashing.

## Hash collisions — what happens when two keys hash to the same slot

```python
# Conceptually: hash("apple") and hash("grape") might land on the same underlying array index
# Python's dict handles this internally via a technique called open addressing / chaining
# -- not something usually implemented by hand, but genuinely worth knowing it's happening,
# since collisions are why hashing is "average case O(1)" but WORST case can degrade to O(n)
```

Worth knowing conceptually for interview depth, but genuinely not something to implement from scratch day-to-day — Python's built-in `dict`/`set` handle collision resolution internally, well-tested and efficient.

## Two Sum, properly — closing the loop from file 03

```python
def two_sum(nums, target):
    seen = {}   # value -> index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:              # O(1) average lookup
            return [seen[complement], i]
        seen[num] = i
    return []

print(two_sum([2, 7, 11, 15], 9))   # [0, 1]
```

`O(n)` time, `O(n)` space — genuinely the canonical "trade space for time" pattern: instead of nested loops checking every pair (`O(n²)`), a single pass builds a hash map of "what have I seen so far," checking in O(1) whether the needed complement already exists.

## Frequency counting — genuinely one of the most common hashing use cases

```python
from collections import Counter

def most_frequent_char(s):
    counts = Counter(s)
    return counts.most_common(1)[0][0]

print(most_frequent_char("abracadabra"))   # 'a'

# Grouping anagrams -- another classic
def group_anagrams(words):
    groups = {}
    for word in words:
        key = ''.join(sorted(word))       # anagrams share the same sorted-letter signature
        groups.setdefault(key, []).append(word)
    return list(groups.values())

print(group_anagrams(["eat", "tea", "tan", "ate", "nat", "bat"]))
# [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]
```

`Counter` is genuinely a specialized dict subclass built exactly for frequency counting — worth using directly rather than manually incrementing dict values with `if key in d: ... else: ...` boilerplate.

## Sets — hashing without the associated value

```python
s = set([1, 2, 2, 3, 3, 3])
print(s)   # {1, 2, 3} -- duplicates automatically removed, O(n) to build

s.add(4)          # O(1) average
s.remove(1)         # O(1) average
print(2 in s)        # O(1) average membership check

# Set operations -- genuinely useful, often overlooked
a = {1, 2, 3}
b = {2, 3, 4}
print(a & b)   # intersection: {2, 3}
print(a | b)   # union: {1, 2, 3, 4}
print(a - b)   # difference: {1}
```

## Finding duplicates — the hashing solution, contrasted against file 03's naive approach

```python
def has_duplicate_hashed(arr):    # O(n) time, O(n) space
    seen = set()
    for num in arr:
        if num in seen:
            return True
        seen.add(num)
    return False
```

Recap file 03's `has_duplicate_naive` — that was `O(n²)`. This hashed version is `O(n)`. Genuinely the exact same "trade memory for speed" trade seen in Two Sum above — a pattern that reappears constantly enough to be worth recognizing as its own reusable idea, not a one-off trick.

## Using tuples as dict keys — a genuinely useful, sometimes-missed technique

```python
# Lists can't be dict keys (mutable, unhashable) -- but tuples CAN
grid_values = {}
grid_values[(0, 0)] = 'start'
grid_values[(3, 4)] = 'end'
print(grid_values[(0, 0)])   # 'start'
```

Genuinely useful for problems involving 2D grids/coordinates (common in graph/matrix problems, file 09) — representing a `(row, col)` pair as a tuple key lets a hash map track visited cells, distances, or other per-position data in O(1) average lookup.

## The honest limitation — hashing loses ORDER (mostly)

```python
# Python dicts DO preserve insertion order (since Python 3.7+), but hashing itself
# doesn't naturally support "give me the smallest key" or "give me keys in sorted order"
# efficiently -- that's what trees (file 07) and heaps (file 08) are for instead
```

Genuinely worth flagging honestly: hashing wins hard for "does this exist" / "how many of this" / "look this up by key" questions, but is the WRONG tool when a problem needs sorted access, range queries, or "find the minimum/maximum efficiently while the data changes" — those needs point toward file 07 (Trees) and file 08 (Heaps) instead.

## Try this

```python
# Given two strings, determine if one is a "rotation" of the other
# (e.g. "waterbottle" and "erbottlewat" -- rotating waterbottle gives erbottlewat)
# Hint: a genuinely elegant O(n) solution uses the fact that if s2 is a rotation of s1,
# then s2 must be a substring of (s1 + s1) -- combine this with Python's `in` check
# Separately: solve the "first non-repeating character in a string" problem using a
# frequency-counting hash map (Counter), in O(n) time
```

Both of these are genuinely common interview questions that become nearly trivial once hashing (and, for the first one, a clever string-doubling trick) click properly — good confirmation this file's core idea has landed.

## What's next
File 07 — Trees, moving from "flat" structures (arrays, linked lists, stacks, queues, hash tables) into hierarchical ones — genuinely a big mental shift, and the foundation for a huge fraction of "medium/hard" interview problems.
