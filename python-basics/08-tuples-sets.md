# 08 — Tuples & Sets

## TUPLES
Ordered, **immutable** collections. Once created, you cannot add, remove, or change elements.

### Creating tuples
```python
t = (1, 2, 3)
single = (5,)        # note the comma — without it, (5) is just an int in parentheses
empty = ()
mixed = (1, "two", 3.0)
```

### Indexing and slicing (same as lists)
```python
t = (10, 20, 30)
t[0]        # 10
t[-1]       # 30
t[0:2]      # (10, 20)
```

### Why use tuples instead of lists?
- Immutability = safer when you don't want data accidentally changed
- Slightly faster than lists (less overhead)
- Can be used as dictionary keys (lists cannot, since keys must be immutable/hashable)
```python
locations = {(0, 0): "origin", (1, 1): "diagonal"}    # tuple as dict key
```

### Tuple unpacking
```python
point = (3, 4)
x, y = point           # x = 3, y = 4

a, b, c = 1, 2, 3        # this is actually tuple packing/unpacking under the hood
```

### Tuples ARE immutable, but...
If a tuple contains a mutable object (like a list), that inner object can still be modified.
```python
t = (1, [2, 3])
t[1].append(4)      # works! -> (1, [2, 3, 4])
t[0] = 100            # ERROR -> can't reassign tuple elements directly
```

### Common tuple methods (limited, since immutable)
```python
t = (1, 2, 2, 3)
t.count(2)      # 2   -> counts occurrences
t.index(3)       # 3   -> index of first occurrence
```

---

## SETS
Unordered collections of **unique** elements — no duplicates allowed. Mutable, but elements themselves must be immutable/hashable.

### Creating sets
```python
s = {1, 2, 3}
s2 = set([1, 2, 2, 3])     # {1, 2, 3}  -> duplicates automatically removed
empty_set = set()             # NOTE: {} creates an empty DICT, not a set
```

### Why sets matter for LeetCode
Sets give O(1) average-time membership checking (`in` operator), much faster than checking `in` on a list (O(n)). Extremely common for "have I seen this before?" problems.
```python
seen = set()
for num in [1, 2, 3, 2]:
    if num in seen:
        print(f"{num} is a duplicate")
    seen.add(num)
```

### Common set methods
```python
s = {1, 2, 3}
s.add(4)             # adds an element
s.remove(2)            # removes element, ERRORS if not present
s.discard(10)             # removes if present, does NOT error if missing
s.pop()                     # removes and returns an arbitrary element
```

### Set operations (mathematical set theory)
```python
a = {1, 2, 3}
b = {2, 3, 4}

a | b     # {1, 2, 3, 4}     -> union
a & b     # {2, 3}            -> intersection
a - b     # {1}                -> difference (in a but not b)
a ^ b     # {1, 4}              -> symmetric difference (in either, not both)

a.issubset(b)        # is a fully contained in b?
a.issuperset(b)        # does a fully contain b?
```

### Sets are unordered
You cannot index into a set (`s[0]` is invalid) since there's no guaranteed order.

### frozenset — immutable version of a set
```python
fs = frozenset([1, 2, 3])     # can't add/remove, but can be used as a dict key
```
